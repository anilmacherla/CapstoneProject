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
		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:638941221632:cluster/EKS-Infra
					'''
				}
			}
		}
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'demo-ecr-credentials') {
					sh '''
						kubectl apply -f ./blue-service.json
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
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}
}
}
