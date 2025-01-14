pipeline {
    agent any

    environment {
        REACT_APP_VERSION="1.0.$BUILD_ID"
        APP_NAME = 'learnjenkinsapp'
        AWS_DEFAULT_REGION = 'us-west-2'
        AWS_DOCKER_REGISTRY = '273354648989.dkr.ecr.us-west-2.amazonaws.com'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
        // NETLIFY_SITE_ID = '5643e5b4-05c8-4407-a65c-a62782a64269'
        // NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    

    stages {
        
        stage('Build') {
            agent{
                docker{ 
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage('Build Docker Image'){
            agent{
                docker{
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {

                    sh '''
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    
                    '''
                }
            }
        }
        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'my-aws-cli'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }
            // environment{
            //     AWS_S3_BUCKET = 'learn-jenkins-757'
            // }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version 
                        sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                        yum install jq -y
                        # aws s3 sync build s3://$AWS_S3_BUCKET
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
                
        }
        // stage('Tests'){
        //     parallel{
        //         stage('Unit tests'){
        //             agent{
        //                 docker{
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }
        //             steps{
        //                 sh ''' 
        //                     #test -f build/index.html
        //                     npm test
        //                 '''
        //             }
        //             post{
        //                 always{
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('E2E'){
        //             agent{
        //                 docker{
        //                     image 'my-playwright'
        //                     reuseNode true
                    
        //                 }
        //             }
        //             steps{
        //                 sh ''' 
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //             post{
        //                 always{
        //                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
        //                 }
        //             }
        //         }
        //     }
        // }
        // stage('Deploy staging'){
        //     agent{
        //         docker{
        //             image 'my-playwright'
        //             reuseNode true
             
        //         }
        //     }
        //     environment{
        //         CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
        //     }
        //     steps{
        //         sh ''' 
        //             netlify --version
        //             echo "Deploying to staging. Site ID: "$NETLIFY_SITE_ID  
        //             netlify status
        //             netlify deploy --dir=build --json > deploy-output.json
        //             CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post{
        //         always{
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }
        // // stage('Approval'){
        // //     steps{
        // //         timeout(time: 15, unit: 'MINUTES'){
        // //             input message: "Do you wish to deploy to production?", ok: "yes, I am sure!"
        // //         }
                
        // //     }
            
        // // }
        // stage('Deploy prod'){
        //     agent{
        //         docker{
        //             image 'my-playwright'
        //             reuseNode true
             
        //         }
        //     }
        //     environment{
        //         CI_ENVIRONMENT_URL = 'https://amazing-dusk-529d06.netlify.app'
        //     }
        //     steps{
        //         sh ''' 
        //             node --version
        //             netlify --version
        //             echo "Deploying to production. Site ID: "$NETLIFY_SITE_ID  
        //             netlify status
        //             netlify deploy --dir=build --prod
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post{
        //         always{
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }
    
        
    }
    
    
}
