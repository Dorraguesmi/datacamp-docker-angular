pipeline {
    agent any

    environment {
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage ('Clone Stage') {
            steps {
                checkout scm
                echo "Version: ${env.DOCKER_TAG}"
            }
        }

        stage ('Docker Build') {
            steps {
                sh '''
cat > Dockerfile << DOCKERFILE
### STAGE 1: Build ###
FROM node:12.7-alpine AS build
WORKDIR /usr/src/app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

### STAGE 2: Run ###
FROM nginx:1.17.1-alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=build /usr/src/app/dist/aston-villa-app /usr/share/nginx/html
DOCKERFILE
'''
                sh 'docker build -t dorra1/angular-app:${DOCKER_TAG} .'
            }
        }

        stage ('DockerHub Push') {
            steps {
                withCredentials([string(credentialsId: 'mydockerhubpassword', variable: 'DockerHubPassword')]) {
                    sh 'docker login -u dorra1 -p ${DockerHubPassword}'
                }
                sh 'docker push dorra1/angular-app:${DOCKER_TAG}'
            }
        }

        stage ('Deploy') {
            steps {
                sshagent(credentials: ['Vagrant_ssh']) {
                    sh 'ssh -o StrictHostKeyChecking=no admin@10.15.122.48 "sudo docker pull dorra1/angular-app:${DOCKER_TAG}"'
                    sh 'ssh -o StrictHostKeyChecking=no admin@10.15.122.48 "sudo docker ps -q | xargs -r sudo docker stop || true"'
                    sh 'ssh -o StrictHostKeyChecking=no admin@10.15.122.48 "sudo docker run -d -p 80:80 dorra1/angular-app:${DOCKER_TAG}"'
                }
            }
        }

    }
}
