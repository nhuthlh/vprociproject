pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        SNAP-REPO = 'nhuthlh-snapshot'
        NEXUS-USER = 'admin'
        NEXUS-PASS = 'admin'
        RELEASE-REPO = 'nhuthlh-release'
        CENTRAL-REPO = 'nhuthlh-maven-central'
        NEXUSIP = '172.31.9.117'
        NEXUSPORT = '8081'
        NEXUS-GRP-REPO = 'vpro-maven-group'
        NEXUS-LOGIN = 'nexuslogin'
    }

    stages {
        stage("Build") {
            steps {
                sh 'mvn -s settings.xml -Dskiptests install'
            }
        }
    }
}