pipeline {
  agent any

  //Configure the following environment variables before executing the Jenkins Job
  environment {
    IntegrationFlowID = "rb_ci_it_test"
	  CPIHost = "${env.CPI_HOST}"
	  CPIOAuthHost = "${env.CPI_OAUTH_HOST}"
	  CPIOAuthCredentials = "${env.CPI_OAUTH_CRED}"	
  }

  stages {
    stage('Generate oauth bearer token') {
      steps {
        script {
          //get oauth token for Cloud Integration
          println("requesting oauth token");
		      try{
            def getTokenResp = httpRequest httpProxy: 'http://rb-proxy-sl.rbesz01.com:8080',acceptType: 'APPLICATION_JSON',
              authentication: env.CPIOAuthCredentials,
              contentType: 'APPLICATION_JSON',
              httpMode: 'POST',
              responseHandle: 'LEAVE_OPEN',
              timeout: 30,
              url: 'https://' + env.CPIOAuthHost + '/oauth/token?grant_type=client_credentials';
            def jsonObjToken = readJSON text: getTokenResp.content
            def token = "Bearer " + jsonObjToken.access_token
            env.token = token
            getTokenResp.close();
		      } catch (Exception e) {
            error("Requesting oauth token for Cloud Integration failed:\n${e}")
          }
        }
      }
    }

    stage('Undeploy integration artefact') {
      steps {
        script {
          //undeploy integration flow as specified in the configuration
          println("Undeploying integration flow.");
		      try{
			      def undeployResp = httpRequest httpProxy: 'http://rb-proxy-sl.rbesz01.com:8080',httpMode: 'DELETE',
              customHeaders: [
                [maskValue: false, name: 'Authorization', value: env.token]
              ],
              ignoreSslErrors: true,
              timeout: 30,
              url: 'https://' + env.CPIHost + '/api/v1/IntegrationRuntimeArtifacts(\'' + env.IntegrationFlowID + '\')';
		      } catch (Exception e) {
            error("Undeploying the integration flow failed:\n${e}")
          }
        }
      }
    }
  }
}
