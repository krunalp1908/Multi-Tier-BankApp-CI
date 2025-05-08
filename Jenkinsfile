@Library('Shared') _

pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'Sonar'
    }

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: '', description: 'Docker image tag for the push')
    }

    stages {

        stage("Validate Parameters") {
            steps {
                script {
                    if (params.DOCKER_TAG.trim() == '') {
                        error("DOCKER_TAG must be provided.")
                    }
                }
            }
        }

        stage("Git Checkout") {
            steps {
                script {
                    code_checkout("https://github.com/krunalp1908/Multi-Tier-BankApp.git", "main")
                }
            }
        }

        stage("Compile") {
            steps {
                sh 'mvn compile'
            }
        }

        stage("Testing") {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage("Trivy: Filesystem Scan") {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage("SonarQube Code Analysis") {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=GCBank \
                        -Dsonar.projectKey=GCBank \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('Publish Artifact'){
            steps{
                withMaven(globalMavenSettingsConfig: 'my-settings-kp', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                     sh "mvn deploy"
                }
            }
        }
        
        stage("Build") {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage("Docker: Build Image") {
            steps {
                script {
                    docker_build("bankapp", params.DOCKER_TAG, "krunalp19")
                }
            }
        }

        stage("Trivy: Image Scan") {
            steps {
                script {
                    try {
                        sh "trivy image --format table -o dimage.html krunalp19/bankapp:${params.DOCKER_TAG}"
                    } catch (err) {
                        echo "Trivy scan failed: ${err}"
                    }
                }
            }
        }

        stage("Docker: Push to Docker Hub") {
            steps {
                script {
                    docker_push("bankapp", params.DOCKER_TAG, "krunalp19")
                }
            }
        }
    }
    post {
    success {
        script {
            build job: 'Bankapp-CD',
                parameters: [
                    string(name: 'DOCKER_TAG', value: params.DOCKER_TAG)
                ],
                propagate: false
            }
        }
    }
}
