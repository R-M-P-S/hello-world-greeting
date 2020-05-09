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
	        "target": "example-project/${BUILD_NUMBER}/",
	        "props": "Integration-Tested=Yes;Performance-Tested=No"
        }
      ]
    }"""
    server.upload(uploadSpec)
  }
  stash name: "binary", includes: "target/hello-0.0.1.war,src/pt/Hello_World_Test_Plan.jmx"
}

node ('docker_pt') {

  stage ('Start Tomcat') {
    sh '''
      cd /home/jenkins/tomcat/bin/
      ./startup.sh
    '''
  }
  stage ('Deploy') {
    unstash 'binary'
    sh '''
      sudo chmod -R 777
      cp target/hello-0.0.1.war home/jenkins/tomcat/webapps/
    '''
  }
  stage ('Performance Testing') {
    sh '''
      cd /opt/jmeter/bin/
      ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l $WORKSPACE/test_report.jtl
    '''

    step([$class: 'ArtifactArchiver',archiveArtifacts: '**/*.jtl'])
  }
  stage ('Promote Build in Artifactory') {
    WithCredentials([usernameColonPassword(credentialsId: 'artifactory-account', variable: 'credentials')]) {
      sh 'curl -u${credentials} -X PUT "http://http://192.168.56.103:8081/artifactory/api/storage/example-project/{BUILD_NUMBER}/hello-0.0.1.war?properies=Performance-Tested=Yes"'
    }
  }
}
