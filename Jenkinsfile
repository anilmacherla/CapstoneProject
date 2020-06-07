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

        stage('Deploy green container') {
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
