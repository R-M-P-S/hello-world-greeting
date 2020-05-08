node ('docker') {
  stage ('poll') {
    checkout scm
  }
  stage ('Build and Unit test') {
    sh 'mvn clean verify -DskipITs=true';
    junit '**/target/surefire-reports/TEST-*.xml'
    archiveArtifacts 'target/*.war'
  }
  stage ('Static code analisys') {
    sh 'mvn clean verify sonar:sonar -Dsonar.projectName=example-project -Dsonar.projectKey=example.project -Dsonar.projectVersion=$BUILD_NUMBER';
  }
  stage ('Integration Test') {
    sh 'mvn clean verify -Dsurefire.skip=true';
    junit '**/target/failsafe-reports/TEST-*.xml'
    archiveArtifacts 'target/*.war'
  }
  stage ('publish') {
    def server = Artifactory.server 'Default Artifactory Server'
    def uploadSpec = """ {
      "files": [
        {
	        "pattern": "target/hello-0.0.1.war",
	        "target": "helloworld-greeting-project/${BUILD_NUMBER}/"
	        "props": "Integration-Tested=Yes;Performance-Tested=No"
        }
      ]
    }"""
    server.upload(uploadSpecs)
  }
}
