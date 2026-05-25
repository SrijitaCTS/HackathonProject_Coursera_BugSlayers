pipeline {
    agent any

    tools {
        // These names must match what you configure in Jenkins → Global Tool Configuration
        jdk 'JDK11'
        maven 'Maven3'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '=== Pulling latest code from GitHub ==='
                checkout scm
            }
        }

        stage('Tool Verification') {
            steps {
                echo '=== Verifying Java and Maven ==='
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Build') {
            steps {
                echo '=== Compiling project (no tests) ==='
                sh 'mvn clean compile -B'
            }
        }

        stage('Test') {
            steps {
                echo '=== Running tests in headless Chrome ==='
                sh 'mvn test -B -Dbrowser=chrome -Dheadless=true'
            }
        }
    }

    post {
        always {
            echo '=== Publishing test reports ==='

            // Surefire / TestNG JUnit-format results
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'

            // Archive Cucumber + Extent + Allure raw outputs as build artifacts
            archiveArtifacts artifacts: 'reports/**/*, target/surefire-reports/**/*, logs/**/*, screenshots/**/*',
                             allowEmptyArchive: true,
                             fingerprint: false
        }
        success {
            echo '✅ Build SUCCESS — all tests passed'
        }
        failure {
            echo '❌ Build FAILED — check console output and archived reports'
        }
        unstable {
            echo '⚠️  Build UNSTABLE — some tests failed'
        }
    }
}