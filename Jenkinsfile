@Library('Shared') _

pipeline {
    agent any
    
    environment {
        // Update the main app image name to match the deployment file
        DOCKER_IMAGE_NAME = 'divyasatpute/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'divyasatpute/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github-credentials')
        GIT_BRANCH = "master"
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/divyasatpute/tws-e-commerce-app.git'
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            try {
                                echo "Building Docker image: ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} using Dockerfile: Dockerfile"
                                
                                // Call docker.build function
                                docker.build(
                                    imageName: "${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}",
                                    dockerfile: 'Dockerfile',
                                    context: '.'
                                )

                                echo "Docker image ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} built successfully."
                            } catch (Exception e) {
                                currentBuild.result = 'FAILURE'
                                echo "Error during Docker build for image ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}: ${e.getMessage()}"
                                echo "Stacktrace: ${e.getStackTrace()}"
                                throw e
                            }
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            try {
                                echo "Building Docker migration image: ${env.DOCKER_MIGRATION_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} using Dockerfile: scripts/Dockerfile.migration"
                                
                                // Call docker.build function for migration image
                                docker.build(
                                    imageName: "${env.DOCKER_MIGRATION_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}",
                                    dockerfile: 'scripts/Dockerfile.migration',
                                    context: '.'
                                )

                                echo "Docker migration image ${env.DOCKER_MIGRATION_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} built successfully."
                            } catch (Exception e) {
                                currentBuild.result = 'FAILURE'
                                echo "Error during Docker build for migration image ${env.DOCKER_MIGRATION_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}: ${e.getMessage()}"
                                echo "Stacktrace: ${e.getStackTrace()}"
                                throw e
                            }
                        }
                    }
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }
        
        stage('Security Scan with Trivy') {
            steps {
                script {
                    // Create directory for results and run Trivy scan
                    trivy_scan()
                }
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }

                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'satputedivya15@gmail.com'
                    )
                }
            }
        }
    }
}
