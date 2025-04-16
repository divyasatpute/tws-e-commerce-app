@Library('Shared') _

pipeline {
    agent any
    
    environment {
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
                            echo "Building Docker image: ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
                            docker.build("${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}", ".")
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            echo "Building Migration Docker image: ${env.DOCKER_MIGRATION_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
                            docker.build("${env.DOCKER_MIGRATION_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}", "-f scripts/Dockerfile.migration .")
                        }
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Call the 'runUnitTests' method from your shared library
                    runUnitTests()
                }
            }
        }

        stage('Security Scan with Trivy') {
    steps {
        script {
            trivyScan.scan()
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
