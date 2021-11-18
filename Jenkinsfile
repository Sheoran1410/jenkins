
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
properties([pipelineTriggers([githubPush()])])
pipeline {
  agent any  
  environment {
    //put your environment variables
    doError = '0'
    DOCKER_REPO = "421320058418.dkr.ecr.us-east-1.amazonaws.com/system:1.0-SNAPSHOT/${JOB_NAME}"
    AWS_DEFAULT_REGION = "us-east-1"
    CHART_DIR="$JENKINS_HOME/workspace/helm-integration/helm"
    HELM_RELEASE_NAME = "api-service"
    ENV= """${sh(
  		returnStdout: true,
  		script: declare -n ENV=${GIT_BRANCH}_env ; echo $ENV ;
    ).trim()}"""
  }
    options {
        buildDiscarder(logRotator(numToKeepStr: '20')) 
  }   
  stages {
    stage ('Build and Test') {
      steps {
        sh '''
        docker build \
        -t ${DOCKER_REPO}:${BUILD_NUMBER} .
        #put your Test cases
        echo 'Starting test cases'
        '''    
      }
    }  
    stage ('Artefact') {
      steps {
        sh '''
        $(aws ecr get-login --region us-east-1 --no-include-email)
        docker push ${DOCKER_REPO}:${BUILD_NUMBER}
        '''
        }
    }   
    stage ('Deploy') {
      steps {
        sh '''
        declare -n CLUSTER_NAME=sheoran-new2-cluster
        aws eks --region $AWS_DEFAULT_REGION  update-kubeconfig --name ${CLUSTER_NAME}
        kubectl config set-context --current --namespace=kube-system
        helm upgrade --install ${HELM_RELEASE_NAME} ${CHART_DIR}/${HELM_RELEASE_NAME}/ \
        --set image.repository=${DOCKER_REPO} \
        --set image.tag=${BUILD_NUMBER} \
        -f ${CHART_DIR}/${HELM_RELEASE_NAME}/values.yaml \
        --namespace
        '''
      }
    }
    stage('Cleanup') {
      steps{
        sh "docker rmi ${DOCKER_REPO}:${BUILD_NUMBER}"
      }
  }
}
