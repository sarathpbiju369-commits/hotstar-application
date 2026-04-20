pipeline{
    agent any
    tools{
        jdk 'jdk'
        nodejs 'node'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
               checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Hotstar \
                    -Dsonar.projectKey=Hotstar '''
                }
            }
        }
         
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
       
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey  F95F0EB1-69BF-F011-8364-0EBF96DE670D', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
           }
        }
            stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build"){
    steps{
        script{
            sh "docker build -t sarathblas/hotstar ."
        }
    }
}
      stage("TRIVY Image Scan"){
    steps{
        sh "trivy image sarathblas/hotstar:latest > trivyimage.txt"
    }
}
       stage("Docker Push"){
    steps{
        script{
            withDockerRegistry([credentialsId: 'docker', toolName: 'docker']) {
                sh "docker push sarathblas/hotstar:latest"
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
