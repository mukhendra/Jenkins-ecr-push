pipeline {
agent any
environment {
AWS_ACCOUNT_ID="723959547348"
AWS_DEFAULT_REGION="us-west-1"
REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
CLUSTER_NAME="jenkins-test-repo"
SERVICE_NAME="jenkins-test-repo"
TASK_DEFINITION_NAME="jenkins-test-repo"
DESIRED_COUNT="2"
IMAGE_REPO_NAME="jenkins-test-repo"
IMAGE_TAG="${env.BUILD_ID}"
registryCredential = "demo-admin-user"
}

stages {

stage('Logging into AWS ECR') {
steps {
script {
sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
}

}
}

stage('Cloning Git') {
steps {
checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/ajeetraina/webpage.git']]])
}
}

// Building Docker images
stage('Building image') {
steps{
script {
dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
}
}
}

// Uploading Docker images into AWS ECR
stage('Pushing to ECR') {
steps{
script {
sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
}
}
}
stage('Deploy') {
  steps{
        withAWS(credentials: registryCredential, region: "${AWS_DEFAULT_REGION}") {
                script {
			sh './script.sh'
                }
            } 
        }
      }      
}
}
