// In this CI/CD setup, the Jenkins pipeline automates the deployment process for a React and Node.js project hosted on GitHub.
// Triggered by GitHub webhook on a push event, Jenkins builds the React frontend, transfers it to an AWS EC2 instance, and serves it using Nginx.
// Simultaneously, the Node.js backend is Dockerized, pushed to Docker Hub, and then pulled and run on the EC2 machine. 
// This streamlined workflow ensures efficient continuous integration and delivery, allowing for rapid updates and deployments in response to changes in the GitHub repository.

// Required Plugins: SSH Agent plugin, node js, Docker pipeline
// Create Credentials: dockerhub, github
pipeline {
    agent any
    tools {
        nodejs "Node-21"
    }
    environment {
        // Define AWS EC2 details
        EC2_HOST = '65.1.248.118'
        EC2_USER = 'ubuntu'
        PRIVATE_KEY = '/var/lib/jenkins/nginx-keypair.pem'
        
        // Define Docker Hub details
        DOCKER_IMAGE_NAME = 'divyapatel42/jenkins-backend-project'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from your Git repository
               script {
                    def tag = 'your_tag_here'
                    
                    // Checkout the specific tag
                    checkout([$class: 'GitSCM', branches: [[name: "refs/tags/v32"]], userRemoteConfigs: [[url: 'https://github.com/Divya4242/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC.git']]])
                    // git branch: 'main', url: 'https://github.com/Divya4242/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC.git', credentialsId: 'github-id' // Provide your Git credentials ID here
                    // git branch: 'main', url: 'https://github.com/Divya4242/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC.git', ref: 'refs/tags/v0.1.0', timeout: 30
                    //sh "curl -LJO -H 'Authorization: token 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' https://api.github.com/repos/Divya4242/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC/releases/tags/v0.1.0/assets/v0.1.0.tar.gz"
                   // sh "wget -q --show-progress --auth-no-challenge --header='Accept:application/octet-stream' https://api.github.com/repos/Divya4242/Divya4242/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC/releases/tags/v0.1.0/assets/v0.1.0.zip?access_token=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"
                }
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
                      // Optional: Use rsyn to copy the entire folder to the EC2 instance.
                    sh "rsync -avrx -e 'ssh -i ${PRIVATE_KEY} -o StrictHostKeyChecking=no' --delete /var/lib/jenkins/workspace/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC/client/build ${EC2_USER}@${EC2_HOST}:/var/www"                  
                }
            }
        }
        
        stage('Build and push Docker Image') {
            steps {
                script {
                     // Build Docker image for Node Backend
                    sh 'whoami'
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:nodebackend", " ./server")
                    docker.withRegistry( '', 'docker-id' ) {  
                        dockerImage.push("nodebackendd")
                    }
                }
            }
        }        
        
        stage('Run Docker Image on AWS EC2') {
            steps {
                script {
                    // This command will delete any contianer running on 5000 so this new docker container run easily.
                    def commands = """
                        docker rmi -f divyapatel42/jenkins-backend-project || true
                        docker rm -f \$(docker ps -q --filter "publish=5000/tcp")
                        docker run -d -p 5000:5000 divyapatel42/jenkins-backend-project:nodebackendd
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
