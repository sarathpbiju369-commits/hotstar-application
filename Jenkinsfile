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
        git branch: 'main', url: 'https://github.com/sarathpbiju369-commits/hotstar-application.git'
    }
}
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

  stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            script {
                def scannerHome = tool 'sonar-scanner'
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=hotstar \
                -Dsonar.projectName=hotstar \
                -Dsonar.sources=.
                """
            }
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
        script {
            def buildStatus = currentBuild.currentResult
            def buildUser = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')[0]?.userId ?: 'Github User'
            
            emailext (
                subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>This is a Jenkins HOTSTAR CICD pipeline status.</p>
                    <p>Project: ${env.JOB_NAME}</p>
                    <p>Build Number: ${env.BUILD_NUMBER}</p>
                    <p>Build Status: ${buildStatus}</p>
                    <p>Started by: ${buildUser}</p>
                    <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: 'sarathpbiju369@gmail.com',
                from: 'sarathpbiju369@gmail.com',
                replyTo: 'sarathpbiju369@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            )
           }
       }

    }

}
