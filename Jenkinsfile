node('jenkins-slave') {
	try {
	container('jenkins-slave'){
		def mvnHome = tool 'maven3'
		def project = "poccrmacr.azurecr.io"
		def appName = "atlas-app"
 		def imageTag = "${project}/${appName}:${env.BRANCH_NAME}"
 		echo 'debut ...'
		
		stage('Push image') {
			sh 'docker images'
			sh 'docker pull hello-world'
			sh 'docker tag hello-world:latest poccrmacr.azurecr.io/hello:${env.BUILD_NUMBER}'
			// sh 'docker push poccrmacr.azurecr.io/hello:1.0'
			
			/* Finally, we'll push the image with two tags:
         		 * First, the incremental build number from Jenkins
         		 * Second, the 'latest' tag.
         		 * Pushing multiple tags is cheap, as all the layers are reused. */
        		docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            			//app.push("${env.BUILD_NUMBER}")
    				//app.push("latest")
				sh 'docker push  poccrmacr.azurecr.io/hello:${env.BUILD_NUMBER}'
        		}
    		} 
		
		
		
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

