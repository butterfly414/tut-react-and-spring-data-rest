pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.9'
        jdk 'JDK17'
    }
    
    
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                          branches: [[name: 'main']],
                          extensions: [],
                          userRemoteConfigs: [[url: 'https://github.com/butterfly414/tut-react-and-spring-data-rest.git']]])
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    script {
                        if (fileExists('target/surefire-reports/TEST-*.xml')) {
                            junit 'target/surefire-reports/TEST-*.xml'
                        } else {
                            echo "Aucun rapport de test trouvé. Vérifiez l'exécution des tests."
                        }
                    }
                }
            }
        }

      /*stage('Code Coverage') {
            steps {
                sh 'mvn org.jacoco:jacoco-maven-plugin:prepare-agent test'
                sh 'mvn org.jacoco:jacoco-maven-plugin:report'
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
        }*/
        
        stage('Code Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle pmd:pmd spotbugs:spotbugs'
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
                sh 'mvn package'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'mvn deploy'
            }
        }

      stage('Deploy to Nexus') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                // Configuration du settings.xml pour Nexus
                configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    sh """
                        mvn -s %MAVEN_SETTINGS% deploy:deploy-file \
                        -Durl=http://localhost:8082//repository/maven-releases \
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
