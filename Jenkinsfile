@NonCPS
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.6.3'
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
                sh './mvnw clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh './mvnw test'
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

        stage('Analyse statique (Checkstyle + PMD)') {
            steps {
                sh './mvnw checkstyle:checkstyle pmd:pmd'
                // Les résultats se trouvent dans target/site
            }
        }

        stage('Documentation Maven Site') {
            steps {
                sh './mvnw site'
                publishHTML(target: [
                    reportDir: 'target/site',
                    reportFiles: 'index.html',
                    reportName: 'Documentation Projet'
                ])
            }
        }
        
        stage('Package') {
            steps {
                sh './mvnw package'
            }
        }
        
        stage('Deploy') {
            steps {
                sh './mvnw deploy'
            }
        }

      /*stage('Deploy to Nexus') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                // Configuration du settings.xml pour Nexus
                configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    sh """
                        mvn -s %MAVEN_SETTINGS% deploy:deploy-file \
                        -Durl=http://localhost:8082/repository/maven-releases \
                        -DrepositoryId=nexus \
                        -Dfile=target/*.jar \
                        -DpomFile=pom.xml \
                        -DskipTests
                    """
                }
            }
        }*/
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
