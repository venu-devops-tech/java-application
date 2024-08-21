pipeline {
    agent any
    tools {
        maven 'maven'
//        jdk "OracleJDK8"
    }

    environment {
        SNAP_REPO = "vprofile-snapshot"
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vprofile-maven-central"
        NEXUSIP = "172.31.21.161"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "vprofile-maven-group"
        NEXUS_LOGIN = "nexuslogin"
        SONARSERVER = 'SonarCloud'
        SONARSCANNER = 'sonarscanner'    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                   echo "Archiving Now"
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

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=venu-devops-tech_java-application \
                    -Dsonar.projectName=java-application \
                    -Dsonar.organization=venu-devops-tech \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
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
        // stage("upload Artifacts") {
        //     steps {
        //         nexusArtifactUploader(
        //            nexusVersion: 'nexus3',
        //            protocol: 'http',
        //            nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        //            groupId: 'QA',
        //            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        //            repository: "$RELEASE_REPO",
        //            credentialsId: "${NEXUS_LOGIN}",
        //            artifacts: [
        //              [artifactId: 'vproapp',
        //               classifier: '',
        //               file: 'target/vprofile-v2.war',
        //               type: 'war']
        //              ]
        //          )
        //     }
        // }
    }
}  