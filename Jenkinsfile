pipeline {
    agent none
    stages {
        stage('Release project processes') {
            agent {
                docker {
                    image 'maven:3-alpine'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            stages {
                stage('Build project') {
                    steps {
                        sh 'mvn -B -DskipTests clean package'
                    }
                }
                stage('Test project') {
                    steps {
                        sh 'mvn test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }
            }
        }
        stage('SonarQube analysis') {
            agent any
            environment {
                sonarqubeScannerHome = tool 'sonar'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://sonarqube:9000 -Dsonar.scm.exclusions.disabled=true -Dsonar.projectName=hello-java -Dsonar.projectVersion=${env.BUILD_NUMBER} -Dsonar.projectKey=javaci -Dsonar.sources=src/main/java/com/mycompany/app"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Publish artifact') {
            agent any
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'nexus:8081',
                    groupId: 'com.mycompany.app',
                    version: '1.0',
                    repository: 'maven-releases',
                    credentialsId: 'nexus',
                    artifacts: [
                        [artifactId: 'javaApp',
                        classifier: '',
                        file: 'target/my-app-1.0.jar',
                        type: 'jar']
                    ]
                )
            }
        }
    }
}