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
		
		stage('Build & Push to Dockerfile') {
                 steps {
                    script {
                                echo "Build Docker Image"
                                dockerImage = docker.build("anilmacherla/capstone:latest")
                                echo "Push Docker Image"
                                retry(2){
                                docker.withRegistry('',"dockerhub" ) {
                                    dockerImage.push()
                                    }
                                }
                            }
                        }
                    }

        stage('Create conf file cluster') {
			        steps {
				        withAWS(region:'us-east-1', credentials:'demo-ecr-credentials') {
					    sh '''
						aws eks --region us-west-2 update-kubeconfig --name Eks-Cluster
					    '''
				    }
			    }
		}
		
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./blue-controller.yaml
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./green-controller.yaml
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./blue-service.yaml
					'''
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
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./green-service.yaml
					'''
				}
			}
		}

	
        

	}
}
