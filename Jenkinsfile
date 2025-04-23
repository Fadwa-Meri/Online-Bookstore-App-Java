pipeline {
    agent any
    tools {
        maven 'Maven 3.9.9'
        jdk 'JDK17'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', 
                url: 'https://github.com/Fadwa-Meri/Online-Bookstore-App-Java.git'
            }
        }
        
        stage('Build') {
            steps {
                bat 'mvn clean compile'
            }
        }
        
        stage('Unit Tests') {
            steps {
                bat 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Code Coverage') {
            steps {
                bat 'mvn cobertura:cobertura'
            }
            post {
                always {
                    cobertura coberturaReportFile: 'target/site/cobertura/coverage.xml'
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                bat 'mvn checkstyle:checkstyle pmd:pmd findbugs:findbugs'
            }
            post {
                always {
                    checkstyle canComputeNew: false, defaultEncoding: '', pattern: '**/target/checkstyle-result.xml'
                    pmd canComputeNew: false, defaultEncoding: '', pattern: '**/target/pmd.xml'
                    findbugs canComputeNew: false, defaultEncoding: '', pattern: '**/target/findbugsXml.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                bat 'mvn package'
            }
        }
        
        stage('Deploy to Nexus') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                bat 'mvn deploy'
            }
        }
    }
    post {
        failure {
            emailext body: 'Build failed: ${BUILD_URL}', 
                     subject: 'Build Failed: ${JOB_NAME} - Build #${BUILD_NUMBER}', 
                     to: 'yasmine.ben-bari@esi.ac.ma'
        }
        unstable {
            emailext body: 'Build unstable: ${BUILD_URL}', 
                     subject: 'Build Unstable: ${JOB_NAME} - Build #${BUILD_NUMBER}', 
                     to: 'yasmine.ben-bari@esi.ac.ma'
        }
    }
}
