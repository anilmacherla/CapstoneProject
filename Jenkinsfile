pipeline {
	agent any

   	environment {
        DOCKER_IMAGE_NAME = "anilmacherla/capstone"
	}

	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build Docker Image') {
            		steps {
                		script {
                    			app = docker.build(DOCKER_IMAGE_NAME)
                    			app.inside {
                        			sh 'echo Hello, Nginx!'
                    			}
                		}
            		}

		}

       		 stage('Push Docker Image') {
            		steps {
                		script {
                    			docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        		app.push("${env.BUILD_NUMBER}")
                        		app.push("latest")
                    			}
                		}
            		}
        	}


        	stage('Deploy blue & Green container') {
            		steps {
                          sshagent(['Project']) {
                             sh "scp -o StrictHostKeyChecking=no  blue-controller.yaml green-controller.yaml blue-service.yaml ubuntu@ec2-34-217-66-56.us-west-2.compute.amazonaws.com:/home/ec2-user/"
                             script{
                                try{
	                            sh "ssh ubuntu@ec2-34-217-66-56.us-west-2.compute.amazonaws.com sudo kubectl apply -f ."
	                     }catch(error){
	                            sh "ssh ubuntu@ec2-34-217-66-56.us-west-2.compute.amazonaws.com sudo kubectl create -f ."
                                          }
                            }
                         }
            	   }
        	}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

                stage('Create the service in the cluster, redirect to green') {
                        steps {
                          sshagent(['Project']) {
                             sh "scp -o StrictHostKeyChecking=no  green-service.yaml ubuntu@ec2-34-217-66-56.us-west-2.compute.amazonaws.com:/home/ec2-user/run/"
                             script{
                                try{
	                            sh "ssh ubuntu@ec2-34-217-66-56.us-west-2.compute.amazonaws.com sudo kubectl apply -f ."
	                     }catch(error){
	                            sh "ssh ubuntu@ec2-34-217-66-56.us-west-2.compute.amazonaws.com sudo kubectl create -f ."
                                          }
                            }
                         }
                        }
                }

	}
}
