def sourcepath = "C:/ProgramData/jenkins/.jenkins/workspace/scmWebApp/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"


pipeline 
{
    environment
    {
        appName = "Farhana-WebApp"
        resourceGroup = "Training-rg"
       
    }
    agent any	
	stages 
	{
        stage('Sonarqube Static Code') 
        {
            steps
            {
                script
                {
                    
                    def scannerHome = tool 'SonarScanner for MSBuild'
                    withSonarQubeEnv('SonarQubeServer') 
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"WebApp\""
                        bat "dotnet build ${sourcepath}"    
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
	/*stage('SonarQube analysis') {
            steps {
                	withSonarQubeEnv('SonarQube') {
                    sh "./gradlew sonarqube"
                	}
           	  }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }*/
        /*stage("Quality gate") 
        {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube'
            }
        }*/
        stage('Pipeline Build')
        {
            steps
            {
                bat "dotnet build ${sourcepath}"
		    
            }
        }
        stage('Testing')
        {
            steps
            {
                bat "dotnet test ${sourcepath}"
		   
            }
        }
        stage('Publishing')
        {
            steps
            {
                bat "dotnet publish ${sourcepath}"
		    
            }
        }
        
        stage('Packaging Stage') 
        {
            steps 
            {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"
                bat "tar.exe -a -c -f WebApp_${BUILD_NUMBER}.zip aspnet-core-dotnet-core/bin/Debug/netcoreapp1.1/publish"
             }
        }
        
        stage ('Connecting to JFrog artifactory')
        {
            steps
            {
               rtServer (
                 id: "Artifactory",
                 //url: 'http://localhost:8082/artifactory',
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }
        stage('Uploading to JFrog')
        {
            steps
            {
                rtUpload (
                 serverId:"Artifactory" ,
                  spec: '''{
                   "files": [
                      {
                      "pattern": "*.zip",
                      "target": "Farhana-Dotnet-App/"
                      }
                            ]
                           }''',
                        )
            }
        }
        stage ('Publish Build Info') 
        {
            steps 
            {
                rtPublishBuildInfo ( serverId: "Artifactory" )
            }
        }
        
        stage ('Downloading artifacts')
        {
            steps
            {
                  rtDownload (
                    serverId: "Artifactory",
                        spec:
                              """{
                                "files": [
                                  {
                                    "pattern": "Farhana-Dotnet-App/WebApp_${BUILD_NUMBER}.zip",
                                    "target": "G:/jfrog/"          
                                  }
                               ]
                              }""" )
              }
        }
      /*  stage('Extract ZIP') 
	      {
	          steps
	          {
		              powershell '''
		                      Expand-Archive 'G:/jfrog/dotnetapp.zip' -DestinationPath 'G:/demo/'
		                  '''
            } 
	      }*/
    
	      stage('Deploying to webapp in azure') 
	      {
	          steps
		        {
		   
			          azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'AzureID', resourceGroup: "${env.resourceGroup}"
	           }
       	}
	}
}
