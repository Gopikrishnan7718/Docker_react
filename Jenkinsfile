pipeline {

    agent any
       environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
       }

    stages {

        stage('Build docker image') {
            steps {
                sh 'docker build -t gopi/docker-react -f Dockerfile.dev .'
            }
        }
        stage('Run tests') {
            steps {
                sh 'docker run -e CI=true gopi/docker-react npm run test'
            }
        }
        stage('deploy') {
            /*when {
                anyOf {
                  branch 'master'
                  branch 'main'
                }
                
            }
            */
            
            steps {

                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-jenkins-creds']]) {
                
                sh '''

                zip -r app.zip .

                aws s3 cp app.zip s3://elasticbeanstalk-ap-south-1-382876614364/docker_react/app.zip

                aws elasticbeanstalk create-application-version --application-name docker_react --version-label v-$BUILD_NUMBER --source-bundle S3Bucket=elasticbeanstalk-ap-south-1-382876614364,S3Key=docker_react/app.zip

                aws elasticbeanstalk update-environment --environment-name Dockerreact-env-1 --version-label v-$BUILD_NUMBER
                
                '''
                }

            }
        }
    } 
}