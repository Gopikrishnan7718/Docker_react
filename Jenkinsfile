pipeline {

    agent any
       environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
       }

    stages {

        stage('Build docker image') {
            steps {
                sh 'docker build -t gopi/docker-react-dev -f Dockerfile.dev .'
            }
        }
        stage('Run tests') {
            steps {
                sh 'docker run -e CI=true gopi/docker-react-dev npm run test'
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

                      # 🔥 WAIT LOGIC (THIS IS WHAT YOU ADD)
                            for i in {1..30}; do
                                status=$(aws elasticbeanstalk describe-application-versions \
                                --application-name docker_react \
                                --version-label v-$BUILD_NUMBER \
                                --query "ApplicationVersions[0].Status" \
                                --output text)

                                echo "Status: $status"

                                if [ "$status" = "PROCESSED" ]; then
                                echo "Version is ready"
                                break
                                fi

                                sleep 10
                            done

                            # Optional safety check
                            if [ "$status" != "PROCESSED" ]; then
                                echo "Timeout waiting for version processing"
                                exit 1
                            fi

                aws elasticbeanstalk update-environment --environment-name Dockerreact-env --version-label v-$BUILD_NUMBER
                
                '''
                }

            }
        }
    } 
}