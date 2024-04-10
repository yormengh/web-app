COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    
    tools {
        maven "maven3.9.6"
    }
    environment {
        ScannerHome = tool "sonar5.0"
    }

    stages {
        stage("Git Clone") {
            steps {
                git branch: 'main', url: 'https://github.com/yormengh/web-app.git'
            }
        }
        stage("Build with Maven") {
            steps {
                sh "mvn clean"
            }
        }
        stage("Test with Maven") {
            steps {
                sh "mvn test"
            }
        }
        stage("Package with Maven") {
            steps {
                sh "mvn package"
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=webapp-project -Dsonar.projeckName=yormengh-job"
                    }
                }
            }
        }
        stage("SonarQube Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Upload to Nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/Webapp-Jenkinsfile-Pipeline-Job/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '3.146.107.255:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-snapshot', version: '3.1.2-SNAPSHOT'
            }
        }
        stage("Deploy to Tomcat") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-access', path: '', url: 'http://18.117.242.180:8080/')], contextPath: null, war: 'target/*.war'
            }
        }
    }
    post {
        always {
            slackSend channel: 'team9-africa', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }
}
