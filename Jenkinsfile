pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "star-banking"
        DOCKER_TAG = "latest"
        DOCKER_REGISTRY = "pravinkr11"
        MAVEN_PATH = sh(script: 'which mvn', returnStdout: true).trim()
        CONTAINER_IMAGE = "${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
        SONARQUBE_SERVER = 'SonarQube' // 👈 replace with your actual server name from Jenkins config
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']], 
                    userRemoteConfigs: [[
                        url: 'https://github.com/devopsbypravin/star-agile-banking-finance.git',
                        credentialsId: 'git_cred'
                    ]]]
                )
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh '''
                        ${MAVEN_PATH} clean verify sonar:sonar \
                        -Dsonar.projectKey=star-banking \
                        -Dsonar.host.url=http://192.168.191.56:9000\
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Compile with Maven') {
            steps {
                sh '''
                    set -e
                    ${MAVEN_PATH} compile
                '''
            }
        }

        stage('Test with Maven') {
            steps {
                sh '''
                    set -e
                    ${MAVEN_PATH} test
                '''
            }
        }

        stage('Install with Maven') {
            steps {
                sh '''
                    set -e
                    ${MAVEN_PATH} install
                '''
            }
        }

        stage('Package with Maven') {
            steps {
                sh '''
                    set -e
                    ${MAVEN_PATH} clean package
                    ${MAVEN_PATH} compile
                    ${MAVEN_PATH} test
                    ${MAVEN_PATH} package
                    ${MAVEN_PATH} install
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e
                    docker build -t ${CONTAINER_IMAGE} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dockerhub_cred', url: 'https://index.docker.io/v1/']) {
                        sh '''
                            set -e
                            docker push ${CONTAINER_IMAGE}
                        '''
                    }
                }
            }
        }
    }
}

