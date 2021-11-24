pipeline{
	agent any
	stages {
	    stage('gitclone') {
			steps {
			    sh 'rm -rf jenkins-test'
				checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/Sheoran1410/jenkins-test']]])
			}
		}
		
    	stage('Build') {
			steps {
				sh 'docker build -t sheoranaks99/npipeline:latest .'
				sh 'docker image list'
			}
		}
		
	    stage('Login and Image Push') {
			steps {
			   withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubpwd', usernameVariable: 'dockerhubusr')]) {
			        sh 'docker login -u ${dockerhubusr}  -p ${dockerhubpwd}'
			        sh 'docker push sheoranaks99/npipeline:latest'
			    }
			}
		}
		
	   	stage('Build remove') {
			steps {
				sh 'docker rmi sheoranaks99/npipeline:latest'
			}
		}
	   
	    stage('Git Clone Helm Chart') {
	        steps{
	            sshagent(credentials:['sheoran-jenkins']) {
	                sh 'rm -rf npipeline'
	                kubernetesDeploy(kubeconfigId: "mykubeconfig")
	                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/Sheoran1410/npipeline-helm.git']]])
                    sh 'ls -la'
                    sh 'pwd'
                    script {
                        try{
                            echo 'try'
                            sh 'helm delete npipeline'
                        } catch(error){
                            echo 'Error Catch'
                        }finally {
                            echo 'finally'
                            sh 'helm install npipeline .'
                        }
	                }
	            }
	        }
	    }
	}
}
