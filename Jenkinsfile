node('jenkins-slave') {
	try {
	container('jenkins-slave'){
		def mvnHome = tool 'maven3'
		def project = "cisnake"
		def appName = "atlas-ms1"
 		// def imageTag = "${project}/${appName}:${env.BRANCH_NAME}"
		def imageTag = "${project}/${appName}:${env.BUILD_NUMBER}"
		def sonarProjectName = "Spring-Boot-Rest-exemple.${env.BRANCH_NAME}"
 		echo 'debut ...'
		
		//stage('Push image') {
		//	sh 'docker images'
		//	sh 'docker pull hello-world'
		//	sh 'docker tag hello-world:latest poccrmacr.azurecr.io/hello:1.0'
		//	// sh 'docker push poccrmacr.azurecr.io/hello:1.0'
		//	
		//	/* Finally, we'll push the image with two tags:
         	//	 * First, the incremental build number from Jenkins
         	//	 * Second, the 'latest' tag.
         	//	 * Pushing multiple tags is cheap, as all the layers are reused. */
        	//	docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            	//		//app.push("${env.BUILD_NUMBER}")
    		//		//app.push("latest")
		//		sh 'docker push poccrmacr.azurecr.io/hello:1.0'
        	//	}
    		//} 
		
		
		stage('Checkout') {
			echo 'Pulling... ' + env.BRANCH_NAME
			checkout scm
			echo 'END Pulling SCM'
			pom = readMavenPom file: 'pom.xml'

			//def project1 = new XmlSlurper().parseText(readFile('pom.xml'))
			echo 'END Pulling SCM 1' 
			//def pomv = project1.version.text()
			echo 'END Pulling SCM ---------------------' + pom.version
		}

		stage('Build') {
			
			echo 'Start Build...1'
			
			// sleep 600
			
			sh "${mvnHome}/bin/mvn install -DskipTests"
			echo 'END Build......'
			
		}

		//stage('Unit Test') {
		//	sh "${mvnHome}/bin/mvn test"
		//}

		//stage('SonarQube analysis') {
		//	withSonarQubeEnv('sonar') {
		//		sh "${mvnHome}/bin/mvn sonar:sonar -Dsonar.projectName=${sonarProjectName}"
		//	}
		//}

		//stage('Quality Gate'){
		//	timeout(time: 30, unit: 'MINUTES') {
		//		def qg = waitForQualityGate()
		//		if (qg.status != 'OK') {
		//			error "Pipeline aborted due to quality gate failure: ${qg.status}"
		//		}
		//	}
		//}

		stage('Build Docker Image') {
			sh "docker build -t ${imageTag} ."

			//app = docker.build("${imageTag}")
		}
		
		stage('Push image') {
			/* Finally, we'll push the image with two tags:
         		 * First, the incremental build number from Jenkins
         		 * Second, the 'latest' tag.
         		 * Pushing multiple tags is cheap, as all the layers are reused. */
        		docker.withRegistry('https://registry.hub.docker.com', 'cisnakeDockerHubId') {
            			//app = docker.build("${imageTag}")
				//app.push()
    				//app.push("latest")
				sh "docker push ${imageTag}"
        		}
    		} 

		//stage('Test image') {
		//	/* Ideally, we would run a test framework against our image.
		//	 * For this example, we're using a Volkswagen-type approach ;-) 
		//	 */
		//	app.inside {
		//		sh 'echo "Tests passed"'
		//	}
		//}

		stage('deploy APP') {
			echo 'Deploy APP from branch ... ' + env.BRANCH_NAME
			def namespace
			if (env.BRANCH_NAME == 'master') {
     				echo 'Deploy to Stagging Environnement'
     				namespace = "staging"
     				
   			} else {
     				echo 'Deploy to Qualif Environnement' + env.BRANCH_NAME
     				namespace = "${env.BRANCH_NAME}".toLowerCase()
     				
		   	}
		   	// Create namespace if it doesn't exist
        		sh("kubectl get ns ${namespace} || kubectl create ns ${namespace}")
			sh("sed -i.bak 's#IMAGE_TAG#${imageTag}#' ./k8s/*.yaml")
			sh("kubectl apply -n ${namespace} -f ./k8s")
		}
		
		if (env.BRANCH_NAME == 'master') {
     			echo 'Deploy to Production Environnement ....'
     			echo 'waiting for approval ...'
     			input message: 'Approve Production deployment ?'
     			stage('PROD Deploy') {
     				namespace = "production"
     				sh("sed -i.bak 's#IMAGE_TAG#${imageTag}#' ./k8s/*.yaml")
				sh("kubectl apply -n ${namespace} -f ./k8s")
     			}		
   		} 
	//}
	}
	} catch (e) {
       		// If there was an exception thrown, the build failed
	    	currentBuild.result = "FAILED"
		echo 'END Build......' + e
        	throw e
   	} finally {
     		// Success or failure, always send notifications
	    	//notifyBuild(currentBuild.result)
   	}
}

def getDevVersion() {
    def gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def versionNumber;
    if (gitCommit == null) {
        versionNumber = env.BUILD_NUMBER;
    } else {
        versionNumber = gitCommit.take(8);
    }
    print 'build  versions...'
    print versionNumber
    return versionNumber
}

def getReleaseVersion() {
    def pom = readMavenPom file: 'pom.xml'
    def gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def versionNumber;
    if (gitCommit == null) {
        versionNumber = env.BUILD_NUMBER;
    } else {
        versionNumber = gitCommit.take(8);
    }
    return pom.version.replace("-SNAPSHOT", ".${versionNumber}")
}

def notifyBuild(String buildStatus = 'STARTED') {
	
	// build status of null means successful
   	buildStatus =  buildStatus ?: 'SUCCESSFUL'
 	   	
	def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
   	def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
     		<p>Check console output at "<a href="${env.BUILD_URL}">${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>"""
   	def mailRecipients = "amine9@gmail.com"
   	
	def myProviders ; //= [ [$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider'] ];

	if (buildStatus == 'SUCCESSFUL') {
     		echo 'amine1'
		myProviders = [ [$class: 'CulpritsRecipientProvider'] ];
   	} else {
     		echo 'amine2'
		myProviders = [ [$class: 'DevelopersRecipientProvider'] ];

   	}
	
	emailext (
           subject: subject,
           body: details,
           to: "${mailRecipients}",
	       recipientProviders: myProviders
     )
}

