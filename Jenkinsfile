node('maven') {
    def dccount
	def applicationname='ENTER APPLICATION NAME'; 
	def buildNumber='1';
	def templatepath =""
	def configMapPropertyFilePath= ""
	def secretPropertyFilePath= ""
	def NAMESPACE=''; // OpenHSift Namespace
	
	stage ('Git Checkout'){
	     // your GIT URL
	    git credentialsId: 'ICP-git', url: ''
	}

	stage('build and deploy app though template'){	
		echo "build the application with maven in maven container provided by openshift"
		//building the application with maven build written witin template ( using strategy docker, maven build is written in dockerfile)
		
	    println "Namespace from paraneter:  ${NAMESPACE}"
		
		sh" oc get dc ${applicationname} -n ${NAMESPACE} | wc -l > deploymentconfigcount"       
		dccount = readFile('deploymentconfigcount')
		
	    println "Is deploying for First time : ${dccount.trim() == '0'}"
	    
	    if(dccount.trim() == '0'){
	        sh """
				#create configmap from file 
				echo "deploy appliction first time through template"
				oc create cm ${applicationname} -n ${NAMESPACE} --from-env-file=${configMapPropertyFilePath}				
				oc label cm ${applicationname} -n ${NAMESPACE} app=${applicationname}
				
				#create secret from file 
				oc create secret generic ${applicationname} -n ${NAMESPACE} --from-env-file=${secretPropertyFilePath}
				oc label secret ${applicationname} -n ${NAMESPACE} app=${applicationname}
				
				#create new app from source code using strategy dockerfile
			    oc new-app -f ${templatepath} -p NAME=${applicationname} -n ${NAMESPACE}  --source-secret=git-cred
    	    """
			//verify Build Status
			verifyBuild(buildNumber?.trim(), applicationname,  NAMESPACE)
			
			//verify deployment
    	    verifyDeployment(buildNumber?.trim(), applicationname,  NAMESPACE)
	    }
	    else{
				
            sh """
				oc delete cm ${applicationname} -n ${NAMESPACE} 
				oc create cm ${applicationname} --from-env-file=${configMapPropertyFilePath} -n ${NAMESPACE}   		
    	        oc label cm ${applicationname} -n ${NAMESPACE} app=${applicationname}
    	        oc start-build ${applicationname} -n ${NAMESPACE} 
    	    """   
			
			buildNumber = sh returnStdout: true ,script: "oc get bc ${applicationname} -n ${NAMESPACE}  -o yaml | grep -w lastVersion | tail -1 | cut -d ':' -f2" 
			
			println "buildNumber..............${buildNumber}"
				
				
			//verify Build Status
			verifyBuild(buildNumber?.trim(),applicationname,  NAMESPACE)
			
			//verify deployment
    	    verifyDeployment(buildNumber?.trim(), applicationname,  NAMESPACE)
	    } 
	}
}


def verifyBuild(String buildNumber, String applicationname, String NAMESPACE) {
    
	//verify Build
    try{
        timeout(5) {
            waitUntil {
                def buildStatus = sh returnStdout: true ,script: "oc get builds  ${applicationname}-${buildNumber} -n ${NAMESPACE}  -o yaml | grep -w phase | tail -1 | cut -d ':' -f2"
             
               
                println "buildStatus .....  ${buildStatus?.trim()}"
                
                return buildStatus?.trim() == 'Complete';
                //return actualReplicas?.trim() == '0';
            }
        }
    }catch(e){
		println "Exception while validating Build status: ${e}"
        println "Timeout while validating Build Status."
        currentBuild.result = "UNSTABLE"
    }

}


def verifyDeployment(String buildNumber, String applicationname, String NAMESPACE) {
    //verify deployment
    try{
        timeout(2) {
            waitUntil {
                def desiredReplicas = sh returnStdout: true ,script: "oc get rc ${applicationname}-${buildNumber}  -n ${NAMESPACE} -o yaml | grep -w replicas | tail -1 | cut -d ':' -f2"
                def actualReplicas = sh returnStdout: true ,script: "oc get rc ${applicationname}-${buildNumber}  -n ${NAMESPACE} -o yaml | grep -w availableReplicas | tail -1 | cut -d ':' -f2"
               
                println "actualReplicas .....  ${actualReplicas?.trim() == desiredReplicas?.trim()}"
                
				if(actualReplicas?.trim() == desiredReplicas?.trim()){
					//Tag ImageStream
					sh " oc tag ${applicationname}:latest ${applicationname}:v${buildNumber} -n ${NAMESPACE} "	
					return true;
				}
								
                return false;
                //return actualReplicas?.trim() == '0';				
						
            }
        }
    }catch(e){
		println "Exception while validating deployment status: ${e}"
        println "Timeout while validating deployment status."
        currentBuild.result = "UNSTABLE"
    }
}
