node('jenkins-slave') {
	container('jenkins-slave'){
 		echo 'debut ...'
		
		stage('Push image') {
			sh 'docker images'
			sh 'docker pull hello-world'
			sh 'docker tag hello-world:latest poccrmacr.azurecr.io/hello:${env.BUILD_NUMBER}'
        		docker.withRegistry('poccrmacr.azurecr.io', 'AzureAcrCredentiel') {
            			//app.push("${env.BUILD_NUMBER}")
    				//app.push("latest")
				sh 'docker push  poccrmacr.azurecr.io/hello:${env.BUILD_NUMBER}'
        		}
    		} 
	}
}
