pipeline {
 agent {
        label 'master.tg.com'
    }
    environment {
        IMAGE_NAME = "cdtsbikaner/my8novdockerhub"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                git credentialsId: 'b07ec2a0-c896-4e43-9104-d8f9560eb3b9',
                    url: 'https://github.com/cdtsbikaner/my8novtest.git',
                    branch: 'master'
            }
        }

        stage('Find Dockerfile') {
            steps {
                sh '''
                  echo "Searching Dockerfile..."
                  find . -name Dockerfile
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:$TAG --no-cache  .
                '''
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh '''
                  docker push $IMAGE_NAME:$TAG
                '''
            }
        }
	   stage('Build, Push & Deploy to Kubernetes') {
		steps {
				sh '''
				  echo "Kubernetes Cluster Status..."
				  kubectl get all

				  kubectl delete svc mytg-svc -n mytg || echo "No such service found , continue ...."

				  echo "Updating image placeholders in YAML..."
				  sed -i "s/BUILD_NUMBER/${TAG}/g" tomcat-deploy.yaml


				  echo "Deploying YAML files..."
				  kubectl apply -f mynps.yaml
				  kubectl apply -f tomcat-deploy.yaml

				  echo "Deployment Status..."
				  kubectl get all -n mytg
				  kubectl get svc -n mytg
				''' 	
			}
		}
    }
	 post {
        always {
            cleanWs()
        }
    }
}
