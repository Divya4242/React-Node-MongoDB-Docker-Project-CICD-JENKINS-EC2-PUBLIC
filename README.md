## In this CI/CD setup, the Jenkins pipeline automates the deployment process for a React and Node.js project hosted on GitHub.
## Triggered by GitHub webhook on a push event, Jenkins builds the React frontend, transfers it to an AWS EC2 instance, and serves it using Nginx.
## Simultaneously, the Node.js backend is Dockerized, pushed to Docker Hub, and then pulled and run on the EC2 machine. 
## This streamlined workflow ensures efficient continuous integration and delivery, allowing for rapid updates and deployments in response to changes in the GitHub repository.

### Required Plugins: SSH Agent plugin, node js, Docker pipeline
### Create Credentials: dockerhub, github
