node ('PASS_CI') 
{
	def previousResult = currentBuild.previousBuild?.result
	//mavenSettingsConfig - Id of jenkins maven settings.xml for the project
	def mavenSettingsConfig = 'XXXX'
	//globalMavenSettingsConfig - Id of jenkins maven global-settings file 
	def globalMavenSettingsConfig = 'XXXX'
	//mavenLocalRepo - folder of the local maven repository
	def mavenLocalRepo = '.repository'
	//profile maven a executer
	def mavenProfiles = 'sonar'


	//checkout from the scm
	stage('checkoutScm')
	{	
		checkout scm
		notifyBuild(currentBuild.result,previousResult)
	}

	//call maven to build the project
	stage('build')
	{
		try 
		{
			withMaven(mavenSettingsConfig: "${mavenSettingsConfig}", globalMavenSettingsConfig: "${globalMavenSettingsConfig}",mavenLocalRepo: "${mavenLocalRepo}" )
		  	{
		    	//launch maven
		    	dir('sources') { sh "mvn clean install -P ${mavenProfiles} -Dmaven.test.failure.ignore=true" }
		  	}
		}
		catch (e) 
		{
			//if we have an exception during the build the result is FAILED
			currentBuild.result = "Failed"
			throw e
		} 
		finally 
		{
			// Success or failure, always send notifications
			notifyBuild(currentBuild.result, previousResult)
		}

	} 

	stage('SonarQube analysis') 
	{
		//we execute sonar only in master
		if (env.BRANCH_NAME == 'master') 
		{
			try 
			{
				def scannerHome = tool 'SonarQube2.8';
				withSonarQubeEnv('SonarQube') 
				{
					dir('sources') { sh "${scannerHome}/bin/sonar-scanner" }
				}
			}
			catch (e) 
			{
				//if we have an exception during the build the result is FAILED
				currentBuild.result = "FAILED"
				throw e
			} 
			finally 
			{
				// Success or failure, always send notifications
				notifyBuild(currentBuild.result, previousResult)
			}
		}
	}
}


/**
* Notification in slack to give the status of the build
* @param currentBuildStatus : status of current build
* @param previousBuildStatus : status of the previous build
*/
def notifyBuild(String currentBuildStatus, String previousBuildStatus)
{
	echo "NotifyBuild [previousBuildStatus:${previousBuildStatus},currentBuildStatus:${currentBuildStatus}]."
  
  	// build status of null means successful
  	currentBuildStatus = currentBuildStatus ?: 'SUCCESS'
  	previousBuildStatus = previousBuildStatus ?: 'SUCCESS'

  	// we set back to normal if we are in success and last wasn't
	if(previousBuildStatus != 'SUCCESS' && currentBuildStatus == 'SUCCESS')
	{
		currentBuildStatus = 'Back to normal'
	}

	//notification text
	def jobName = java.net.URLDecoder.decode("${env.JOB_NAME}", "UTF-8");
	def subject = "${jobName} - #${env.BUILD_NUMBER} ${currentBuildStatus}"
  	def summary = "${subject} (<${env.BUILD_URL}|Open>)"

  	//colors
  	if (currentBuildStatus == 'STARTED' || currentBuildStatus == 'UNSTABLE') 
	{
		//Yellow
		colorCode = '#FFFF00'
	} 
  	else if (currentBuildStatus == 'SUCCESS' || currentBuildStatus == 'Back to normal') 
	{
		//Green
		colorCode = '#00FF00'
	}
	else 
	{
		//Red
		colorCode = '#FF0000'
	}

	// we notify only errors and back to normal
	if(currentBuildStatus != 'STARTED' && currentBuildStatus != 'SUCCESS')
	{
		slackSend (color: colorCode, message: summary)
	}
}