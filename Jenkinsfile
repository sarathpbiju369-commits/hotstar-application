pipeline {
    agent any

    tools {
        jdk 'jdk'
        nodejs 'node'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git 'https://github.com/sarathpbiju369-commits/hotstar-application.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=hotstar \
                    -Dsonar.projectKey=hotstar
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('OWASP FS SCAN') {
            steps {
                sh 'dependency-check.sh --scan . --format XML'
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker', toolName: 'docker']) {
                        sh 'docker build -t sarathblas/hotstar .'
                    }
                }
            }
        }

        stage('TRIVY Image Scan') {
            steps {
                sh 'trivy image sarathblas/hotstar:latest > trivyimage.txt'
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker', toolName: 'docker']) {
                        sh 'docker push sarathblas/hotstar:latest'
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed successfully"
        }
    }
}
