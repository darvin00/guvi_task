def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
        maven 'MAVEN3'
        jdk 'OracleJDK17'
    }
    environment {
        registryCredential = 'ecr:us-west-2:awscreds'
        appRegistry = "905418343282.dkr.ecr.us-west-2.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://905418343282.dkr.ecr.us-west-2.amazonaws.com"
    }
    stages {
        stage('Fetch Code'){
            steps {
                git branch: 'docker', url: 'https://github.com/darvin00/guvi_task.git' 
            }
        }
        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post{
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle' 
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
                withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
               timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build(appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
            }
        }
        stage('Upload App Image') {
            steps{
                script {
                    docker.withRegistry(vprofileRegistry, registryCredential) {
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        success {
            mail to: 'darvinxavier007@gmail.com',
                 subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: """<p>SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                          <p>Check console output at '<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>'</p>"""
        }
        failure {
            mail to: 'darvinxavier007@gmail.com',
                 subject: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                 body: """<p>FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                          <p>Check console output at '<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>'</p>"""
        }
    }
}
