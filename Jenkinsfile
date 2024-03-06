pipeline {
    agent any
    tools {
        nodejs "Node-21"
    }
    environment {
        // Define AWS EC2 details
        EC2_HOST = '65.0.87.100'
        EC2_USER = 'ubuntu'
        PRIVATE_KEY = '/var/lib/jenkins/nginx-keypair.pem'
        
        // Define Docker Hub details
        DOCKER_IMAGE_NAME = 'divyapatel42/jenkins-backend-project'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from your Git repository
                git branch: 'main', url: 'https://github.com/Divya4242/Jenkins-React-Node.git'
            }
        }

        stage('Build React App') {
            steps {
                // Build React app
                sh 'cd client && npm install && npm run build && cd build && pwd'
            }
        }
        
        stage('Transfer Frotend Build to EC2') {
            steps {
                script {
                    // Use scp to copy the entire folder to the EC2 instance
                  sh "scp -i ${PRIVATE_KEY} -r /var/lib/jenkins/workspace/frontend-backend-deploy/client/build ${EC2_USER}@${EC2_HOST}:/var/www"
                  // Optional: Use rsyn to copy the entire folder to the EC2 instance (not working)
                // sh '''
                //   sudo rsync -avz -e "ssh -i ${PRIVATE_KEY} -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" --delete --progress -r /var/lib/jenkins/workspace/frontend-backend-deploy/client/build ${EC2_USER}@${EC2_HOST}:/var/www
                // '''    
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:nodebackend", " ./server")
                }
            }
        }        

        stage('Upload to Docker Hub') {
            steps {
                script {
                    docker.withRegistry( '', 'docker-id' ) {  
                        dockerImage.push("nodebackend")
                    }
                }
            }
        }
        
        stage('Run Docker Image on AWS EC2') {
            steps {
                script {
                    def commands = """
                        docker run -d -p 5000:5000 divyapatel42/jenkins-backend-project:nodebackend
                    """
                    // SSH into EC2 instance and pull Docker image
                    sshagent(['ec2-ssh']) {
                    sh "ssh -o StrictHostKeyChecking=no -i ${PRIVATE_KEY} ${EC2_USER}@${EC2_HOST} '${commands}'"
                    }
                }
            }
        }
    }
        
    post {
        success {
            echo 'Pipeline succeeded. Node.js app deployed to AWS EC2.'
        }
        failure {
            echo 'Pipeline failed. Check the logs for errors.'
        }
    }

}
