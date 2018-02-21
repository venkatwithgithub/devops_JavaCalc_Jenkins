pipeline {
 agent none

 // Reading pom.xml file using readMavenPom(), which is available once we install pipeline utility steps plugin
 //environment { VERSION = readMavenPom().getVersion() }

 // Retention Policy for keeping 2 Builds and 1 Artifact
 options {
  buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
 }

 stages {

  stage('Running on Node1') {
   agent {
    label 'node1'
   }
   steps {
    //Use the script step and wrap arbitrary pipeline script inside of it
    script {
     //Reading pom.xml file using readMavenPom(), which is available once we install pipeline utility steps plugin
     env.VERSION = readMavenPom().getVersion()
    }

    echo 'This is running on Node1'
    echo 'Publishing the Artficat: ' + VERSION
    sh 'hostname'

   }
  }

  // Running Unit tests and Generate test reports  
  stage('Unit Tests') {

   // This agent is for specifying on which node the build shoould get executed, now test cases will run on master
   agent {
    label 'node1'
   }

   steps {
    sh 'mvn test'
   }
   post {
    success {
     junit 'target/surefire-reports/*.xml'
    }
   }

  }

  // Packaging the Application  
  stage('Build') {

   agent {
    label 'node1'
   }

   steps {
    // Application will be packaged on node1
    echo 'Building Calculator'
    sh 'mvn package'
   }
  }

  /* Publish the generated artifacts to Nexus Repository Manager
  For this we need to install Nexus plugin(all) and configure nexus in Configure System
  nexusPublisher we can get from pipelineSyntax, also we need to do In-process Script Approval
  */
  stage('Publish Nexus') {

   agent {
    label 'node1'
   }

   steps {
    echo "$VERSION"
    echo 'Publishing the Artficat: ' + VERSION
    nexusPublisher nexusInstanceId: 'Nexus', nexusRepositoryId: 'releases', packages: [
     [$class: 'MavenPackage', mavenAssetList: [
      [classifier: '', extension: '', filePath: '/home/jenkins/workspace/maven-package/target/RaviCalculator-' + VERSION + '.jar']
     ], mavenCoordinate: [artifactId: 'RaviCalculator', groupId: 'com.ravi.cal', packaging: 'jar', version: VERSION]]
    ]
   }
  }

  stage('Test on Debian') {

   agent {
    // opjdk:version
    docker {
     image 'openjdk:latest'
     label 'node1'
    }
   }

   steps {
    // sh 'wget Cal.jar'
    // sh 'java -jar Cal.jar'
    sh 'wget http://52.77.231.213:8081/nexus/service/local/repositories/releases/content/com/ravi/cal/RaviCalculator/' + VERSION + '/RaviCalculator-' + VERSION + '.jar'
    sh 'java -cp target/RaviCalculator-' + VERSION + '.jar com.ravi.cal.RaviCalculator.Calculator 4 5'
   }
  }

  stage('Promote Dev Branch To Master') {
   agent {
    label 'node1'
   }

   when {
    branch 'dev'
   }
   steps {
    echo 'Stashing Local Changes'
    sh 'git stash'
    echo 'Checking out dev Branch '
    sh 'git checkout dev'
    echo 'Checking out master Branch '
    sh 'git checkout master'
    echo 'Merging dev into master Branch '
    sh 'git merge dev'
    echo 'Pushing into master Branch '
    sh 'git push origin master'


   }

  }

 }

 // Post Build Option --> Archiving Artifacts steps
 /*    post {
     always {
     archiveArtifacts 'target/*.jar'
     }
   }
 */

}
