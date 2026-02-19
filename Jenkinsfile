pipeline {
	agent any
	stages {
		stage ('Build Backend') {
			steps {
				bat 'mvn clean package -DskipTests=true'
			}
		}
		stage ('Unit Test') {
			steps {
				bat 'mvn test'
			}
		}
		stage ('Sonar Analysis') {
			environment {
				scannerHome = tool 'SONAR_SCANNER'
			}
			steps {
				withSonarQubeEnv('SONAR_LOCAL')
				bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=sqa_6c635532f25098fa5890d73fe01726c7771d6acb -Dsonar.allowPermissionManagementForProjectAdmins=true -Dsonar.java.binaries=target -Dsonar.java.libraries=target -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml -Dsonar.coverage.exclusions=**/src/test/**,**/model/**,**/*Application.java -Dsonar.projectVersion=%BUILD_NUMBER%"
			}
		}
	}
}

