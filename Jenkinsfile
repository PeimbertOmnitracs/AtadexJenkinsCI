pipeline {
   agent any
   stages {
      stage('Checkout code') {
         steps {
            dir('sylectus-edi-processor') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', git branch: 'develop'; url: 'https://github.com/Omnitracs/sylectus-edi-processor.git'
            }
            dir('sylectus-log-utils') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-log-utils.git'
            }
            dir('sylectus-lib-sylectus-core') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus-core.git'
            }
            dir('sylectus-lib-sylectus-geo-services') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus-geo-services.git'
            }
            dir('sylectus-lib-sylectus-security-util') {
                git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus-security-util.git'
            }
            dir('sylectus-lib-sylectus') {
               git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus.git'
            }
            //add the repos that sylectus.core depend on
            dir('sylectus-lib-sylectus-security-util'){
               git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: 'https://github.com/Omnitracs/sylectus-lib-sylectus-security-util.git'
            }
            dir('sylectus-lib-sylectus'){
               git credentialsId: '04a6a3f5-a14e-457d-9a94-a61acfcf7e1b', url: ' https://github.com/Omnitracs/sylectus-lib-sylectus.git'
            }
            
            
           
         }
      }
      stage('add hangfire NugetSource'){
        steps{
           // bat label: '', script: '''C:\\ProgramData\\chocolatey\\bin\\nuget.exe sources remove -name "Nuget Hangfire" -source https://nuget.hangfire.io/nuget/hangfire-pro'''
            //bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe sources add -name "Nuget Hangfire Source" -source https://nuget.hangfire.io/nuget/hangfire-pro -username Omnitracs -password BLKE7CvkVSAgnEIB '
            bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe sources list'
        }
      }
      stage('Restore Nuget packages'){
        steps{
            bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-edi-processor/EDIProcessor.sln'  
        }
      }
      stage('Restore dependencie packages'){
         steps{
		   echo '************************** RESTORE DEPENDENCIE PACKAGES *************************' 
           bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus/trunk/SylectusLibrary.sln'
           bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus-security-util/trunk/Sylectus.SecurityUtil.sln'
           bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus-geo-services/trunk/Sylectus.Geoservices.sln'
		   bat label: '', script: 'C:\\ProgramData\\chocolatey\\bin\\nuget.exe restore sylectus-lib-sylectus-core/Sylectus.Core/Sylectus.Core.sln'

         }
      }
      
      
      stage('Pre-build dependencies '){
        steps{
            echo '************************** BUILD DEPENDENCIES *************************' 
            //TARGET net framework v4.6.1 /p:TargetFrameworkVersion=v4.6.1
            //bat "\"${tool 'MSBuildLocal'}\" sylectus-lib-sylectus-security-util/trunk/Sylectus.SecurityUtil.sln /p:Configuration=Release;Platform=\"Any CPU\";AdditionalLibPaths='sylectus-lib-sylectus/trunk/';TargetFrameworkVersion=v4.5.2  /target:rebuild /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
            //bat "\"${tool 'MSBuildLocal'}\" sylectus-lib-sylectus-core/Sylectus.Core/Sylectus.Core.sln /p:Configuration=Release;Platform=\"Any CPU\";TargetFrameworkVersion=v4.6 /target:rebuild /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
			bat "\"${tool 'MSBuildLocal'}\" sylectus-edi-processor/EDIProcessor.sln /p:Configuration=Release;Platform=\"Any CPU\";TargetFrameworkVersion=v4.6 /target:rebuild /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
            bat "\"${tool 'MSBuildLocal'}\" sylectus-edi-processor/EDIAPI/EDIAPI.csproj /p:DeployOnBuild=true /p:PublishProfile=FolderProfile"

            //bat "\"${tool 'MSBuildLocal'}\" sylectus-lib-sylectus/trunk/SylectusLibrary.sln /p:Configuration=Release /p:Platform=\"Any CPU\"   /target:rebuild /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
        }
      }
      

      /*So far the project builds, the next step is to copy the output of the output of EDIAPI publish folder and HangfireRelease Folder and the XML files when changes.
      *then put those into a folder with the date yyyy.mm.dd-Staging
      *Trigger the build with every commit to development.
      */
      
     /*  stage('Build of the solution '){
        steps{
            //TARGET net framework v4.6.1 /p:TargetFrameworkVersion=v4.6.1
            bat "\"${tool 'MSBuildLocal'}\" sylectus-edi-processor/EDIProcessor.sln /p:Configuration=Release /p:Platform=\"Any CPU\"   /target:rebuild /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
        }
      }*/
   }
}
