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
                cleanWs()
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${env.GIT_BRANCH}", url: 'https://github.com/divyasatpute/tws-e-commerce-app.git'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                            docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}", ".")
                        }
                    }
                }

                stage('Build Migration Image') {
                    steps {
                        script {
                            echo "Building Migration Docker image: ${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                            docker.build("${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}", "-f scripts/Dockerfile.migration .")
                        }
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    runUnitTests()
                }
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                script {
                    trivyScan(
                        imageName: "${DOCKER_IMAGE_NAME}",
                        imageTag: "${DOCKER_IMAGE_TAG}",
                        threshold: 100,
                        severity: 'HIGH,CRITICAL'
                    )
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}").push()
                        docker.image("${DOCKER_MIGRATION_IMAGE_NAME}:${DOCKER_IMAGE_TAG}").push()
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
