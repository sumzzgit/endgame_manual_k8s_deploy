pipeline {
    agent any
    environment{
        SEC_KEY = credentials('endgame')
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sumzzgit/endgame_eks_deploy.git'
            }
        }
        stage("build docker images"){
            steps{
                script{
                    withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "aws-creds",
                ]]){
                    sh '''FRONTEND_REPO_URI=$(aws secretsmanager get-secret-value --secret-id endgame --query SecretString --output text | jq -r ".fronted_repo_uri")
                          BACKEND_REPO_URI=$(aws secretsmanager get-secret-value --secret-id endgame --query SecretString --output text | jq -r ".backend_repo_uri")
                          REGION=$(aws secretsmanager get-secret-value --secret-id endgame --query SecretString --output text | jq -r ".region")
                          cd backend
                          docker build -t backend:${BUILD_TAG} .
                    
                          cd ../frontend
                          docker build -t frontend:${BUILD_TAG} -f Dockerfile-modified .
                    
                          docker tag frontend:${BUILD_TAG} ${FRONTEND_REPO_URI}:${BUILD_TAG}
                          docker tag frontend:${BUILD_TAG} ${FRONTEND_REPO_URI}:latest
                    
                          docker tag backend:${BUILD_TAG} ${BACKEND_REPO_URI}:${BUILD_TAG}
                          docker tag backend:${BUILD_TAG} ${BACKEND_REPO_URI}:latest
                    
                          aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin 533267334695.dkr.ecr.ap-south-1.amazonaws.com
                    
                          docker push ${FRONTEND_REPO_URI}:${BUILD_TAG}
                          docker push ${FRONTEND_REPO_URI}:latest
                    
                          aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin 533267334695.dkr.ecr.ap-south-1.amazonaws.com
                    
                          docker push ${BACKEND_REPO_URI}:${BUILD_TAG}
                          docker push ${BACKEND_REPO_URI}:latest
                        '''
                }
            }
        }

        }
    stage("deploy to eks"){
        steps{
                script{
                    withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "aws-creds",
                ]]){
                sh '''
                
                docker rmi -f $(docker images -q)
                
                export SECRET_NAME=$(echo ${SEC_KEY} | jq -r ".secret_name")
                export SVC_ACC_ROLE_ARN=$(echo ${SEC_KEY} | jq -r ".svc_acc_role_arn")
                export BACKEND_REPO_URI=$(echo ${SEC_KEY} | jq -r ".backend_repo_uri")
                export FRONTEND_REPO_URI=$(echo ${SEC_KEY} | jq -r ".fronted_repo_uri")
                export CLUSTER_NAME=$(echo ${SEC_KEY} | jq -r ".cluster_name")
                export REGION=$(echo ${SEC_KEY} | jq -r ".region")
                export FRONTEND_VERSION=${BUILD_TAG}
                export BACKEND_VERSION=${BUILD_TAG}
                
                envsubst < serviceaccount_tmp.yml > serviceaccount.yml 
                envsubst < providerclass_new_tmp.yml > providerclass_new.yml
                envsubst < backend-deploy_tmp.yml > backend-deploy.yml
                envsubst < frontend-deploy_tmp.yml > frontend-deploy.yml
                
                aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${REGION}
                
                kubectl apply -f serviceaccount.yml 
                kubectl apply -f providerclass_new.yml
                kubectl apply -f backend-deploy.yml
                kubectl apply -f backend-svc.yml
                kubectl apply -f frontend-deploy.yml
                kubectl apply -f frontend-svc.yml
            '''
                }
            }
        }
    }
    }
}
