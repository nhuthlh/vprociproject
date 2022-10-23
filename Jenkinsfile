pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        SNAP_REPO = 'nhuthlh-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'nhuthlh-release'
        CENTRAL_REPO = 'nhuthlh-maven-central'
        NEXUSIP = '172.31.9.117'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'nhuthlh-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
    }

    stages {
        stage("Build") {
            steps {
                sh 'mvn -s settings.xml -Dskiptests install'
            }
            post {
                success {
                    echo "Now Archiving..."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage("Test") {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}