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
