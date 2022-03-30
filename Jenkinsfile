def sourcepath = "C:/ProgramData/jenkins/.jenkins/workspace/scmWebApp/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"
def publishpath = "aspnet-core-dotnet-core/bin/Debug/netcoreapp1.1/publish"
def artifactdestinationpath = "G:/jfrog/"


pipeline 
{
   /* environment
    {
        appName = "Farhana-WebApp"
        resourceGroup = "Training-rg"
       
    }*/
    agent any	
	stages 
	{
        stage('Sonarqube Static Code') 		//sonarqube static code analyses stage
        {
            steps
            {
                script
                {
                    
                    def scannerHome = tool 'SonarScanner for MSBuild'		//variable defined to store the installed tool
                    withSonarQubeEnv('SonarQubeServer') 			// This block is used to select the sonarqube server
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"WebApp\"" 
                        bat "dotnet build ${sourcepath}"    					
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
	
        stage("Quality gate") 		//stage used to check quality gate status
        {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }
        stage('Pipeline Build')		// stage used to build the project and all its dependencies.
        {
            steps
            {
                bat "dotnet build ${sourcepath}"	//dotnet build command uses msbuild to build the project so that both parallel and incremental build is supported
		    
            }
        }
        stage('Testing')		//  stage used to execute unit tests in a given solution. 
        {
            steps
            {
                bat "dotnet test ${sourcepath}"		
		   
            }
        }
        stage('Publishing')		//this stage publishes the application and its dependencies to a folder for deployment to a hosting system.
        {
            steps
            {
                bat "dotnet publish ${sourcepath}"
		    
            }
        }
        
        stage('Packaging Stage')  	//stage used to package published files.
        {
            steps 
            {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"
		    bat "tar.exe -a -c -f WebApp_${BUILD_NUMBER}.zip ${publishpath}"
             }
        }
        
        stage ('Connecting to JFrog artifactory')  	//connecting jenkins to jfrog
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
        stage('Uploading to JFrog')		//This allows managing the File Specifications
        {
            steps
            {
                rtUpload (
                 serverId:"Artifactory" ,	//Creating an Artifactory Server Instance
                  spec: '''{
                   "files": [
                      {
                      "pattern": "*.zip",
                      "target": "Farhana-Dotnet-App/"
                      }
                            ]
                           }''',
                        )		//Farhana-Dotnet-App is the repository created in jfrog
            }
        }
        stage ('Publish Build Info') 	//The depedencies and artifacts which are recorded locally is published as build-info to Artifactory using this stage.
        {
            steps 
            {
                rtPublishBuildInfo ( serverId: "Artifactory" )
            }
        }
        
        stage ('Downloading artifacts')		//this stage allows to download the specified files from the artifactory to a destination path.
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
                                    "target": "${artifactdestinationpath}"          
                                  }
                               ]
                              }""" )
              }
        }
      
    
	      stage('Deploying to webapp in azure') 	//stage used to deploy the published files to azure web app
	      {
	          steps
		        {
		   
			          //azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'AzureID', resourceGroup: "${env.resourceGroup}"
				azureWebAppPublish azureCredentialsId: params.Credential_ID , resourceGroup: params.Resource_Group , appName: params.webappName
				
	           }
       	}
	}
}
