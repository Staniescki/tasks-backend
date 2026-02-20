pipeline {
    agent any

    stages {

        stage('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }

        stage('Unit Test') {
            steps {
                bat 'mvn test jacoco:report'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat """
                        "${scannerHome}\\bin\\sonar-scanner.bat" -e ^
                        -Dsonar.projectKey=DeployBack ^
                        -Dsonar.host.url=http://localhost:9000 ^
                        -Dsonar.login=sqa_6c635532f25098fa5890d73fe01726c7771d6acb ^
                        -Dsonar.allowPermissionManagementForProjectAdmins=true ^
                        -Dsonar.java.binaries=target ^
                        -Dsonar.java.libraries=target ^
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml ^
                        -Dsonar.coverage.exclusions=**/src/test/**,**/model/**,**/*Application.java ^
                        -Dsonar.projectVersion=%BUILD_NUMBER%
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                deploy adapters: [
                    tomcat8(
                        alternativeDeploymentContext: '',
                        credentialsId: 'TomcatLogin',
                        path: '',
                        url: 'http://localhost:8001'
                    )
                ],
                contextPath: 'tasks-backend',
                war: 'target\\tasks-backend.war'
            }
        }

        stage('API Tests') {
            steps {
                dir('api-test') {
					deleteDir()
                    git credentialsId: 'github_login',
                        url: 'https://github.com/Staniescki/tasks-api-test'
                    bat 'mvn test'
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git credentialsId: 'github_login',
                        url: 'https://github.com/Staniescki/tasks-frontend'
                    bat 'mvn clean package'

                    deploy adapters: [
                        tomcat8(
                            alternativeDeploymentContext: '',
                            credentialsId: 'TomcatLogin',
                            path: '',
                            url: 'http://localhost:8001'
                        )
                    ],
                    contextPath: 'tasks',
                    war: 'target\\tasks.war'
                }
            }
        }

        stage('Functional Tests') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'github_login',
                        url: 'https://github.com/Staniescki/tasks-functional-test'
                    bat 'mvn test'
                }
            }
        }

        stage('Deploy Prod') {
          steps {
            bat '''
                  wsl bash -lc "export BUILD_NUMBER=%BUILD_NUMBER% && docker compose down --remove-orphans || true"
                  wsl bash -lc "export BUILD_NUMBER=%BUILD_NUMBER% && docker compose up -d --build --remove-orphans"
                  wsl bash -lc "export BUILD_NUMBER=%BUILD_NUMBER% && docker compose ps"
                '''
          }
        }

        stage('Health Check') {
            steps {
            sleep(10)
                dir('functional-test') {
                    bat 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
        }
        unsuccessful {
            emailext attachlog: true, body: 'See the attached log below', subject: 'Build #${env.BUILD_NUMBER} has failed', to: 'dstaniescki@gmail.com'
        }
        fixed {
            emailext attachlog: true, body: 'See the attached log below', subject: 'Build #${env.BUILD_NUMBER} is fine!', to: 'dstaniescki@gmail.com'
        }
    }
}
