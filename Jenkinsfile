// In this CI/CD setup, the Jenkins pipeline automates the deployment process for a React and Node.js project hosted on GitHub.
// Triggered by GitHub webhook on a tag creation event, Jenkins builds the React frontend, transfers it to an AWS EC2 instance, and serves it 
// using Nginx. Simultaneously, the Node.js backend is Dockerized, pushed to Docker Hub, and then pulled and run on the EC2 machine. 
// This streamlined workflow ensures efficient continuous integration and delivery, allowing for rapid updates and deployments in response to 
// changes in the GitHub repository. GITHUB ACTION to create tag when merge happens into main branch. JENKINS for CICD.

// Required Plugins: SSH Agent plugin, node js, Docker pipeline
// Create Credentials: dockerhub, github, ec2-ssh(line no. 77)
pipeline {
    agent any
    
    environment {
        // Define AWS EC2 details
        EC2_HOST = '65.0.173.69'
        EC2_USER = 'ubuntu'
        PRIVATE_KEY = '/var/lib/jenkins/nginx-keypair.pem'
        
        // Define Docker Hub details
        DOCKER_IMAGE_NAME = 'divyapatel42/ecommerce-webapp'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from your Git repository
               script {
                   // if you want to Checkout to tag USE this.
                     // def Specifictag = 'v58' // if you want to checkout to specific tag uncomment this and change to [name: "refs/tags/${Specifictag}"]],.
                    // def latestTag = sh(script: 'git describe --tags $(git rev-list --tags --max-count=1)', returnStdout: true).trim()
                    // checkout([$class: 'GitSCM', branches: [[name: "refs/tags/${Specifictag}"]], userRemoteConfigs: [[url: 'https://github.com/Divya4242/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC.git']]])
                   // if you want to Checkout to branch USE this.
                   git branch: 'main', url: 'https://github.com/Divya4242/React-Node-Docker-Project-CICD-JENKINS-EC2-PUBLIC.git', credentialsId: 'github-id' // Provide your Git credentials ID here
                }
            }
        }

        // stage('Build React App') {
        //     steps {
        //         // Build React app
        //         sh 'cd client && npm install && npm run build && cd build && pwd'
        //     }
        // }
        
        stage('Transfer Frotend to EC2') {
            steps {
                script {
                      // Optional: Use rsyn to copy the entire folder to the EC2 instance.
                    sh "rsync -avrx -e 'ssh -i ${PRIVATE_KEY} -o StrictHostKeyChecking=no' --delete /var/lib/jenkins/workspace/Ecommerce-React-Node-Docker-Project-CICD-JENKINS-Divya/client ${EC2_USER}@${EC2_HOST}:~/reactapp/"                  
                }
            }
        }
        
        stage('Build and push Docker Image') {
            steps {
                script {
                     // Build Docker image for Node Backend
                    sh 'whoami'
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:backendjenkins", " ./server")
                    docker.withRegistry( '', 'docker-id' ) {  
                        dockerImage.push("backendjenkins")
                    }
                }
            }
        }        

        stage('Run Docker Image on AWS EC2') {
            steps {
                script {
                    // This command will delete any contianer running on 5000 so this new docker container run easily.
                    def commands = """
                        cd reactapp/client
                        npm install
                        sudo rm -r build/
                        npm run build
                        sudo rm -r /var/www/build/
                        sudo mv build/ /var/www/
                        cd 
                        docker rmi -f divyapatel42/jenkins-backend-project:nodebackend || true
                        docker rm -f \$(docker ps -q --filter "publish=5000/tcp")
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
