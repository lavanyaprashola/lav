pipeline {
  agent any
  tools {
  maven 'maven'
  }
    stages {

  stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://Maiyappanshivaji@bitbucket.org/maiyappanshivaji/loginapp-cicd-integration.git']]])
      }
   }
	  
	stage ('Build')  {
	    steps {
        dir('app'){
            sh "mvn package"
          }
        }    
   }
   
  stage ('SonarQube Analysis') {
    steps {
      withSonarQubeEnv('sonar') {           
				dir('app'){
          sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=jenkins1052'
        }
    }
    }
 }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "https://awscloudplatform.jfrog.io/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "spidymaven-libs-release-local",
                    snapshotRepo: "spidymaven-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "default-maven-virtual",
                    snapshotRepo: "default-maven-virtual"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "maven", // Tool name from Jenkins configuration
                    pom: 'app/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

stage('Docker Build') {
      steps {
        script {
              docker.withRegistry( 'https://registry.hub.docker.com', 'docker' ) {
              def customImage = docker.build("maiyappan/webapp01")
              customImage.push()
          }
      }
    }
}
   stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

  stage('Build Helm Charts') {
    steps {
        dir('charts') {
        withCredentials([usernamePassword(credentialsId: 'jfrog', usernameVariable: 'username', passwordVariable: 'password')]) {
             sh 'sudo /usr/local/bin/helm package webapp'
             sh 'sudo /usr/local/bin/helm push-artifactory webapp-1.0.tgz https://dtdemoaspire.jfrog.io/artifactory/helmjen-helm-local --username $username --password $password'
		  }
        }
        }
      }

  } 
}