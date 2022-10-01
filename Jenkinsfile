pipeline{
    agent any
    tools{
        maven "Maven"
        jdk   "JDK8"
    }
    stages{   
      stage("fetch from Github"){
         steps{
            git branch: "vp-rem", url: "https://github.com/developer-mide/vprofile-project.git"
         }
         post{
            success{
                echo "Fetched from Github successfully"
            }
            failure{
                echo "Unable to fetch from Github"
            }
          }
       }

    stage("build with maven"){
        steps{
             sh "mvn install -DskipTests"
          }
          post{
             success{
                archiveArtifacts artifacts: "**/target/*.war"
                echo "Build successful"
             }
             failure{
                echo "Unable to build code!"
             }
          }
       }

    stage("run unit tests"){
        steps{
            sh "mvn test"
        }
        post{
            success{
                echo "Great code you have got there"
            }
            failure{
                echo "Unit tests did not go successfully, code needs to be reviewed"
            }
         }        
      }
    stage("code analysis with sonarqube"){
        steps{
            withSonarQubeEnv('sonar'){
              sh "mvn sonar:sonar \
                 -Dsonar.projectKey=new-era-ci-project \
                 -Dsonar.host.url=http://ec2-54-237-201-133.compute-1.amazonaws.com:9000 \
                 -Dsonar.login=31c66a707f6b15905bc7270f36457401c957ac70"
           } 
         }
      }
    stage("wait for sonarqube quality gates"){
        steps{
                timeout(time: 1, unit: "MINUTES"){
                waitForQualityGate abortPipeline: true
           }
        }
        post{
            success{
                nexusArtifactUploader(
                   nexusVersion: 'nexus3',
                   protocol: 'http',
                   nexusUrl: 'ip-172-31-23-213.ec2.internal:8081/repository/new-era-ci',
                   groupId: 'com.example',
                   version: "new-era-project-${env.BUILD_TIMESTAMP}",
                   repository: 'new-era-ci',
                   credentialsId: 'nexus-login',
                   artifacts: [
                     [artifactId: "vprofile-project",
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                   ]
                )
            }
            failure{
                echo "Quality gate returns an error"
            }
        }
     }
   }
}

