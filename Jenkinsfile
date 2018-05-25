node('jenkins-slave') {
	container('jenkins-slave'){
 		echo 'debut ...'
		
		def mvnHome = tool 'maven3'
		def project = "poccrmacr.azurecr.io"
		def appName = "hello-cisnake"
 		def imageTag = "${project}/${appName}:${env.BUILD_NUMBER}"
		
		stage('Push image') {
			sh 'docker images | grep hello'
			//sh 'docker pull hello-world'
			sh 'kubectl get pods -n crmcontract'
			sh ("docker tag hello-world:latest ${imageTag} ")
        		docker.withRegistry('http://poccrmacr.azurecr.io', 'azure-hub-credentials') {
            			//app.push("${env.BUILD_NUMBER}")
    				//app.push("latest")
				sh ("docker push  ${imageTag}")
				// sh("kubectl apply -n ${namespace} -f ./k8s")
        		}
    		} 
	}
}
