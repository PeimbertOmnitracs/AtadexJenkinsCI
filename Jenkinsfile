/** This Jenkins pipeline is to make the build of the Atadex Project from the develop branch. It takes all the commits (doesnt cherry-pick)
   PREREQUISITE: You have to be connected to the VPN so it can copy the files to the CTP Server

   2021.01.06 Robocopy was replaced by xcopy, because Robocopy has exit codes 0 and 1 and thats fine, but Jenkins detects the Exit code 1 as error
*/

pipeline {
   agent any
   stages {
      stage('Checkout code') {
         steps {
            dir('sylectus-edi-processor') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', branch: 'develop', url: 'https://github.com/Omnitracs/sylectus-edi-processor.git'
            }
            dir('sylectus-log-utils') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', branch: 'develop', url: 'https://github.com/Omnitracs/sylectus-log-utils.git'
            }
            dir('sylectus-lib-sylectus-core') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus-core.git'
            }
            dir('sylectus-lib-sylectus-geo-services') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus-geo-services.git'
            }
            //add the repos that sylectus.core depend on
            dir('sylectus-lib-sylectus-security-util'){
               git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus-security-util.git'
            }
            /* --looks like syletus-lib-sylectus is not required for edi-processor
            dir('sylectus-lib-sylectus'){
               git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: ' https://github.com/Omnitracs/sylectus-lib-sylectus.git'
            }*/
            
            
         }
      }
      stage('add hangfire NugetSource'){
        steps{
           echo '************************** LIST NUGET SOURCES *************************' 
           bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe sources list'
        }
      }

      stage('Restore dependencies nuget packages'){
         steps{
		   echo '************************** RESTORE DEPENDENCIE PACKAGES *************************' 
           bat """
           C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-edi-processor/EDIProcessor.sln  
           C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus/trunk/SylectusLibrary.sln
           C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus-security-util/trunk/Sylectus.SecurityUtil.sln
           C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus-geo-services/trunk/Sylectus.Geoservices.sln
		     C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus-core/Sylectus.Core/Sylectus.Core.sln
           """

         }
      }
      
      
      stage('Build the Project'){
        steps{
            echo '************************** BUILD DEPENDENCIES *************************' 
            bat "\"${tool 'MSBuildLocal'}\" sylectus-edi-processor/EDIProcessor.sln /p:Configuration=Release;Platform=\"Any CPU\";TargetFrameworkVersion=v4.6 /target:Clean,Rebuild /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
            bat "\"${tool 'MSBuildLocal'}\" sylectus-edi-processor/EDIAPI/EDIAPI.csproj /p:DeployOnBuild=true /p:PublishProfile=FolderProfile /p:publishUrl=C:/Users/Erick/Documents/testAtadexbuildOutput"

        }
      }
      

      /*So far the project builds, the next step is to copy the output of the output of EDIAPI publish folder and HangfireRelease Folder and the XML files when changes.
      *then put those into a folder with the date yyyy.mm.dd-Staging
      *Trigger the build with every commit to development.
      */
      
      stage('Zipping the artifact '){
        steps{
            echo '************************** ZIPPING THE ARTIFACT *************************' 
            powershell '''
            $DateOfBuild = Get-Date -Format FileDate
            $Env = "Staging"
            $ArtifactName= "ATA" + "$DateOfBuild" +"$Env"
                        
            robocopy.exe "C:\\Program Files (x86)\\Jenkins\\workspace\\atadex\\sylectus-edi-processor\\EDIAPI\\bin\\Release\\Publish"    ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName+"\\EDIAPI\\Publish") /mir /tee
            robocopy.exe "C:\\Program Files (x86)\\Jenkins\\workspace\\atadex\\sylectus-edi-processor\\EDIHangfireServer\\bin\\Release" ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName+"\\EDIHangfireServer\\Release") /mir /tee
            robocopy.exe "C:\\Program Files (x86)\\Jenkins\\workspace\\atadex\\sylectus-edi-processor\\EDIProcessor\\Config"           ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName+"\\Config") /mir /tee
            
            Compress-Archive -Path ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName) -DestinationPath ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName) -Force
            $LASTEXITCODE
            if($LASTEXITCODE -eq 1){
               Exit 0
            }
            
            '''
            
        }
      }        
/*
      stage('Sending the artifact to the CTP Staging Server'){
        steps{
           echo '************************** SENDING THE ARTIFACT TO CTP SERVER *************************' 
            powershell '''
            $DateOfBuild = Get-Date -Format FileDate
            $Env = "Staging"
            $ArtifactName= "ATA" + "$DateOfBuild" +"$Env"            
            robocopy.exe "C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY" \\\\10.231.143.35\\e$\\Deploy\\EDIProcessor ($ArtifactName+".zip") /V /ZB
            
            ####Code to allow robocopy to return exit code 1 without failing the jenkins job
            $LASTEXITCODE
            if($LASTEXITCODE -eq 1){
               Exit 0
            }
            '''
            
        }
      }
      
*/


        




   }
}


/*

bat label: '', script: '''
            $DateOfBuild = Get-Date -Format FileDate
            $Env = "Staging"
            $ArtifactName= "ATA" + "$DateOfBuild" +"$Env"
            
            robocopy.exe "C:\\Program Files (x86)\\Jenkins\\workspace\\atadex\\sylectus-edi-processor\\EDIAPI\\bin\\Release\\Publish"    ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName+"\\EDIAPI\\Publish") /mir /tee
            robocopy.exe "C:\\Program Files (x86)\\Jenkins\\workspace\\atadex\\sylectus-edi-processor\\EDIHangfireServer\\bin\\Release" ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName+"\\EDIHangfireServer\\Release") /mir /tee
            robocopy.exe "C:\\Program Files (x86)\\Jenkins\\workspace\\atadex\\sylectus-edi-processor\\EDIProcessor\\Config"           ("C:\\Users\\Erick\\Documents\\ATADEX FILES FOR DEPLOY\\"+$ArtifactName+"\\Config") /mir /tee'''

*/