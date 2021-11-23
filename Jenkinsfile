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
				sh 'sudo docker build -t sheoranaks99/npipeline:latest .'
			}
		}
		
	    stage('Login and Image Push') {
			steps {
			   withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerhubpwd', usernameVariable: 'dockerhubusr')]) {
			        sh 'sudo docker login -u ${dockerhubusr}  -p ${dockerhubpwd}'
			        sh 'sudo docker push sheoranaks99/npipeline:latest'
			    }
			}
		}
		
	   	stage('Build remove') {
			steps {
				sh 'sudo docker rmi sheoranaks99/npipeline:latest'
			}
		}
	   
	    stage('Git Clone Helm Chart') {
	        steps{
	            sshagent(['instance_k8s']) {
	                sh 'rm -rf npipeline'
	                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/Sheoran1410/npipeline-helm']]])
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
