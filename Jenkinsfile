COLOR_MAP = [
    SUCCESS: 'good', // Changed the colon to a comma
    FAILURE: 'danger', // Changed the colon to a comma
]
pipeline {
    agent any
    tools {
        maven "maven3.9.6" 
    }
    stages {
        stage("Git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/alfredapubuosazure/web-app.git'
            }
        }
        stage("Build with Maven") {
            steps {
                sh "mvn clean"
            }
        }
        stage("Testing with Maven") {
            steps {
                sh "mvn test"
            }
        }
        stage("Package with Maven") { // Corrected the capitalization of "package"
            steps {
                sh "mvn package"
            }
        }
        stage('SonarQube Analysis') {
            environment {
                ScannerHome = tool 'sonarqube5.0'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/second-pipeline-job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '3.83.253.67:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-snapshot', version: '3.1.2-SNAPSHOT'
            }
        } // Added missing closing curly brace
    
        stage("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-credential-', path: '', url: 'http://44.202.39.224:8080/')], contextPath: null, war: 'target/*.war'
            }
        }
    }
    post {
        success {
            slackSend channel: 'team1-africa', color: COLOR_MAP.SUCCESS, message: "Build successful: ${currentBuild.fullDisplayName}" // Changed 'good' to COLOR_MAP.SUCCESS
        }
        failure {
            slackSend channel: 'team1-africa', color: COLOR_MAP.FAILURE, message: "Build failed: ${currentBuild.fullDisplayName}" // Changed 'danger' to COLOR_MAP.FAILURE
        }
    }
}
