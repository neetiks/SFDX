#!groovy
import groovy.json.JsonSlurperClassic

  @NonCPS
    def jsonParse(def json) 
    {
   	 	new groovy.json.JsonSlurperClassic().parseText(json)
	}
	
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def toolbelt = tool 'toolbelt'
    def SCRATCH_ORG_USER_NAME
	
	/*
    def HUB_ORG_USER_NAME=env.HUB_ORG_USER_NAME
    def SFDC_LOGIN_URL = env.SFDC_LOGIN_URL
    def JWT_CRED_ID_FOR_PRIVATE_KEY_FILE = env.JWT_CRED_ID_FOR_PRIVATE_KEY_FILE
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY
    */
    
    def CONNECTED_APP_CONSUMER_KEY="3MVG9d8..z.hDcPLICRMy5HexmHQGA4gSidl5JWH2MtgBPZAWqzjOpxm59alKsd9APAWNJx8djI2IaRbSfrk7"
    def HUB_ORG_USER_NAME="rahul.pathade@cognizant.com"
    def SFDC_LOGIN_URL = "https://login.salesforce.com"
    def JWT_CRED_ID_FOR_PRIVATE_KEY_FILE = "ebab1336-b167-4049-aa78-0529f3db39af"
   

    stage('checkout source') 
    {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_CRED_ID_FOR_PRIVATE_KEY_FILE, variable: 'jwt_key_file')]) 
   { 
        stage('Authorize hub org and set default CI scratch org') 
        {
			
			rc = bat returnStatus: true, script: "${toolbelt}/sfdx' force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG_USER_NAME} --jwtkeyfile server.key --setdefaultdevhubusername --instanceurl ${SFDC_LOGIN_URL}"
           
            if (rc != 0) { error 'hub org authorization failed' }	
            
           // need to pull out assigned username
		   rmsg = sh returnStdout: true, script: "'${toolbelt}/sfdx' force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername"
		   
		   //printf rmsg		   
		    
		   def robj =jsonParse(rmsg)	   
		   		   
		   if (robj.status != 0) { error 'org creation failed: ' + robj.message }
		   
		   //error 'robj.username: ' + robj.toString() + '  '+ robj.result.username+ '  ' + robj.result.orgId
		   
		   SCRATCH_ORG_USER_NAME=robj.result.username
		   
		   robj = null
           
           //assign the default username
          	 
          //SCRATCH_ORG_USER_NAME= env.SCRATCH_ORG_USER_NAME                    	          
                 	
        } 

        stage('Push To Test Org') 
        {
            rc = sh returnStatus: true, script: "'${toolbelt}/sfdx' force:source:push --targetusername ${SCRATCH_ORG_USER_NAME}"
            
            if (rc != 0) 
            {
                error 'push to CI scratch org failed'
            }
            
            // assign permission set
            rc = sh returnStatus: true, script: "'${toolbelt}/sfdx' force:user:permset:assign --targetusername ${SCRATCH_ORG_USER_NAME} --permsetname DreamHouse"
            
            if (rc != 0) 
            {
                error 'permission set assignment failed'
            }
        }

        stage('Run Apex Test') 
        {
            sh "mkdir -p ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') 
            {
                rc = sh returnStatus: true, script: "'${toolbelt}/sfdx' force:apex:test:run --testlevel RunLocalTests --outputdir '${RUN_ARTIFACT_DIR}' --resultformat human --targetusername ${SCRATCH_ORG_USER_NAME}"
                if (rc != 0) 
                {
                    error 'apex test run failed'
                }
            }
        }

        stage('collect results') 
        {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
            
           // rc = sh returnStatus: true, script: "'${toolbelt}/sfdx'  force:apex:test:report -i 707p000000UxELL --resultformat human"
        }
        
        /*
        stage('Open the CI Scratch org') 
        {
            rc = sh returnStatus: true, script: "'${toolbelt}/sfdx' force:org:open --targetusername ${SCRATCH_ORG_USER_NAME}"
            if (rc != 0) 
            {
                error 'Open of CI Scratch org failed'
            }
        }
        */
        stage('Delete Test Org') 
        {

        timeout(time: 120, unit: 'SECONDS') 
        {
         
	        
	           // rc = sh returnStatus: true, script: "'${toolbelt}/sfdx' force:org:delete --targetusername ${SCRATCH_ORG_USER_NAME} --noprompt"
	          //  if (rc != 0) 
	          //  {
	                error 'org deletion request failed'
	         //   }
	        
        }
    }
    }
}
