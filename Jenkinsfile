pipeline {
    agent any
    environment {
        MAVEN_OPTS = "--add-opens jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED"
        TEST_OPTS = "-DskipTests=false -Dmaven.test.skip=false"
        ANALYSIS_OPTS = "-Dcheckstyle.skip=false -Dpmd.skip=false -Dspotbugs.skip=false"
    }
    tools {
        maven 'Maven 3.9.9'
        jdk 'JDK17' 
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: 'master']],
                          extensions: [],
                          userRemoteConfigs: [[url: 'https://github.com/Fadwa-Meri/Online-Bookstore-App-Java.git']]])
            }
        }
        
        stage('Build') {
            steps {
                bat "mvn clean -Dmaven.compiler.release=17"
            }
        }
        
        stage('Unit Tests') {
            steps {
                bat "mvn clean ${TEST_OPTS}"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        
        stage('Code Coverage') {
            steps {
                bat "mvn org.jacoco:jacoco-maven-plugin:prepare-agent test ${TEST_OPTS}"
                bat "mvn org.jacoco:jacoco-maven-plugin:report"
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'target/site/jacoco',
                        reportFiles: 'index.html',
                        reportName: 'JaCoCo Report'
                    ])
                }
            }
        }
        
        stage('Code Analysis') {
            steps {
                bat "mvn checkstyle:checkstyle pmd:pmd spotbugs:spotbugs ${ANALYSIS_OPTS}"
            }
            post {
                always {
                    recordIssues(
                        tools: [
                            checkStyle(pattern: '**/target/checkstyle-result.xml'),
                            pmdParser(pattern: '**/target/pmd.xml'),
                            spotBugs(pattern: '**/target/spotbugsXml.xml', useRankAsPriority: true)
                        ],
                        qualityGates: [[threshold: 1, type: 'NEW', unstable: true]]
                    )
                }
            }
        }
        
        stage('Package') {
            steps {
                bat "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy to Nexus') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                // Configuration du settings.xml pour Nexus
                configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    bat """
                        mvn -s %MAVEN_SETTINGS% deploy:deploy-file \
                        -Durl=${NEXUS_URL}/repository/${NEXUS_REPO} \
                        -DrepositoryId=nexus \
                        -Dfile=target/*.jar \
                        -DpomFile=pom.xml \
                        -DskipTests
                    """
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            cleanWs() 
        }
        failure {
            emailext body: '''Build failed: ${BUILD_URL}
                              |Consultez les logs pour plus de détails.'''.stripMargin(), 
                     subject: 'ÉCHEC Build: ${JOB_NAME} - Build #${BUILD_NUMBER}', 
                     to: 'yasmine.ben-bari@esi.ac.ma',
                     attachLog: true
        }
        unstable {
            emailext body: '''Build unstable: ${BUILD_URL}
                              |Problèmes de qualité de code détectés.'''.stripMargin(), 
                     subject: 'Build Instable: ${JOB_NAME} - Build #${BUILD_NUMBER}', 
                     to: 'yasmine.ben-bari@esi.ac.ma'
        }
    }
}
