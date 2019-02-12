#!groovy
package com.javacodegeeks.example.groovy.json;
import groovy.json.JsonSlurperClassic
import groovy.json.JsonSlurper
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    //def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {

            rc = bat returnStatus: true, script: "sfdx force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            if (rc != 0) { error 'hub org authorization failed' }
            else { print "Successfully Authorized..."}
            

            // need to pull out assigned username
            print "Going to Create Scratch ORG..."
            rmsg = bat returnStdout: true, script: "sfdx force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername"
            //printf rmsg
            //print rmsg
            //print "Scratch Org Created Successfully"
            //def rmsg = '{"status":0,"result":{"orgId":"00DO00000055yi9MAA","username":"test-div6bgju47fs@example.com"}}'
            //def rmsg = ["orgId":"00DO00000055yi9MAA","username":"test-div6bgju47fs"]
            //String rmsg = "{}"
            //print rmsg
            //def jsonSlurper = new JsonSlurper()
            //def robj = jsonSlurper.parseText(rmsg)
            //if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            //SFDC_USERNAME=robj.result.username
            //robj = null
            //def jsonSlurper = new JsonSlurperClassic()
            //def rmsg = '{"orgId":"00DO00000055yi9MAA","username":"test-div6bgju47fs@example.com"}'
            print "Going to print rmsg Value"
            print rmsg
            //def robj = jsonSlurper.parseText(rmsg)
            //print "Checking if username could be parsed"
            //print robj.result.username
            //if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            //SFDC_USERNAME=robj.result.username
            //robj = null
            def jsonSlurper = new JsonSlurper()
            //def inputText = '{"name" : "Groovy", "year": 2005}'
            //def inputText = '{"orgId":"00DO00000055yi9MAA","username":"test-div6bgju47fs@example.com"}'

            def jsonObject = jsonSlurper.parseText(rmsg)
            println "JSONObject generated out of JsonSlurper : " + jsonObject
            println "Username -> [" + jsonObject.result.username + "]"
            SFDC_USERNAME=jsonObject.result.username
        

        }

        stage('Push To Test Org') {
            rc = bat returnStatus: true, script: "sfdx force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'push failed'
            }
            // assign permset
            rc = bat returnStatus: true, script: "sfdx force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname Geolocation"
            if (rc != 0) {
                error 'permset:assign failed'
            }
        }

        stage('Run Apex Test') {
            bat "mkdir -p ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') {
                rc = bat returnStatus: true, script: "sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'apex test run failed'
                }
            }
        }

        stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
        }
    }
}
