def COLOR_MAP = [
    'SUCCESS' : 'good',
    'FAILURE' : 'danger',
    'UNSTABLE': 'warning',
    'ABORTED': 'warning'
]

pipeline {
    agent any
    tools {
        maven "maven"
        jdk "openjdk-11"
    }
    environment {
        SNAP_REPO = 'vproflie-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '192.168.1.14'
        NEXUS_URL = '192.168.1.14'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile-maven-group'

        // Additional Docker variables
        appregistry = "https://registry.hub.docker.com"
        vprofileRegistry = "boopathy102"
        registryCredential = "dockerHub"
        
        DOCKER_IMAGE_NAME = "boopathy102/vprofile"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Boopathy02/vprofile-project.git', branch: 'cicd-jenkins'
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
                withCredentials([usernamePassword(credentialsId: 'nexus3', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${NEXUS_URL}:${NEXUSPORT}",
                        groupId: 'Prod',
                        version: "${env.BUILD_ID}",
                        repository: "${RELEASE_REPO}",
                        credentialsId: 'nexus3',
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
        stage('Build App image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}", "./Docker-files/app/")
                }
            }
        }
        stage('Upload App Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com',registryCredential) {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }
}
