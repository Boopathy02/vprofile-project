pipeline {
    agent any
    tools {
        maven "maven"
        jdk "openjdk-11"
    }
    environment {
        SNAP_REPO = 'vproflie-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'VicK#@344'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '192.168.1.34'
        NEXUS_URL = 'http://192.168.1.34:8081'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile-maven-group'
    }
    
    stages {
        stage ('Checkout') {
            steps {
                git url: 'git@github.com:vanthiyadhevan/vprofile-project.git', branch: 'ci-jenkins', credentialsId: 'vanthiyadhevan'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('Upload Artifact To Nexus') {
            steps {
                    withCredentials([string(credentialsId: 'nexus', variable: '${PASS}')]) {
                            nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "${NEXUS_URL}:${NEXUSPORT}",
                            groupId: 'QA',
                            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                            repository: "${RELEASE_REPO}",
                            credentialsId: 'nexus',
                            artifacts: [
                                [artifactId: 'vproapp',
                                classifier: '',
                                file: 'target/vprofile-v2.war',
                                type: 'war']
                            ]
                        )
                    }
               }
        }
    }
}
