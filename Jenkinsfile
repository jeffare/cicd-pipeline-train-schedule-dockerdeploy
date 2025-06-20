// This is the full recipe for our local deployment pipeline
pipeline {
    // This pipeline can run on any available Jenkins machine
    agent any

    // These are the sequential steps of our automation
    stages {

        // Stage 1: Build the Java application using Gradle
        stage('Build App') {
            steps {
                echo 'Building the Java application...'
               // sh './gradlew build --no-daemon'
                sh "npm install"
            }
        }

        // Stage 2: Create a Docker container image from the app
        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker image...'
                // The 'app' variable will hold our built image
                script {
                    app = docker.build("jeffngara/train-schedule")
                }
            }
        }

        // Stage 3: Push the image to Docker Hub for storage
        stage('Push Docker Image') {
            steps {
                echo 'Pushing the image to Docker Hub...'
                // Use the 'docker_hub_login' credential to log in
                script{
                docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                    // Push two versions: one with a unique build number, and one called 'latest'
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                }
                }
            }
        }

        // Stage 4: Deploy the container to your local machine
        stage('Deploy To Local Machine') {
            steps {
                // This step pauses the pipeline for you to give manual approval
                input 'Ready to deploy to your local machine?'

                echo 'Deploying container locally...'
                script {
                    // Pull the specific image version from Docker Hub to ensure we have the right one
                    sh "docker pull <DOCKER_HUB_USERNAME>/train-schedule:${env.BUILD_NUMBER}"

                    // Stop and remove any old version of this container if it exists
                    // This 'try/catch' block prevents errors if the container isn't already running
                    try {
                        sh "docker stop train-schedule"
                        sh "docker rm train-schedule"
                    } catch (err) {
                        echo 'No old container found. Ready for the new one.'
                    }

                    // Run the new container on your local machine
                    sh "docker run --restart always --name train-schedule -p 9000:8080 -d <DOCKER_HUB_USERNAME>/train-schedule:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
