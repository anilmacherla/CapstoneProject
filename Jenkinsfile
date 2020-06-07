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


        stage('Deploy blue container') {
			steps {
					withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./blue-deployment.yaml
					'''
				}
			}
		}

		stage('Wait user approve') {
            steps {
                	input "Ready to redirect traffic to green?"
            }
        }

        stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./green-deployment.yaml
					'''
				}
			}
		}

	}
}
