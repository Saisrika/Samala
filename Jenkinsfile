def registry = 'https://trialrbzab7.jfrog.io'
pipeline {
    agent any
    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-amazon-corretto.x86_64"
        PATH = "/opt/maven/bin:$PATH"

    }
   
    stages {

       stage("build") {
            steps {
                echo "-------------------build started-----------------"
                sh 'mvn clean install -Dmaven.test.skip=true'
                
                echo "-------------------build Ended-----------------"
            }
       }

               stage("test") {
            steps {
                echo "-------------------Unti Test started-----------------"
                
                sh 'mvn surefire-report:report'
                
                echo "-------------------Unit Test Ended-----------------"
            }
       }

       stage('Sonarqube analysis') {
            environment {
                scannerHome = tool 'sai-sonar-scanner'
            }
            
            steps {
              withSonarQubeEnv('sai-sonar-server') {
                 
                  sh "${scannerHome}/bin/sonar-scanner"
              }
            }
        }
        stage ('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        
                        def qg = waitForQualityGate()
                        
                        if (qg.status != 'OK') {
                            echo "Warning: Quality gate failed but continuing pipeline: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage ("Jar Publish") {
            steps {
                script {
                   echo "<---------------- Jar Publish Started --------------->"
                def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "JFrog-cred"

                def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"

                def uploadSpec = """{
                         "files":[
                    {
                         "pattern": "jarstaging/{*}",
                         "target": "sai-libs-release-local/{1}",
                         "flat": "false",
                         "props": "${properties}",
                         "exclusions": [ "*.sha1", "*.md5"]
                }
              ]
          }"""
          
          def buildInfo = server.upload(uploadSpec)
          
          buildInfo.env.collect()

          server.publishBuildInfo(buildInfo)

              echo "<----------------- Jar Publish Finished ------------------>"
         }
       }
    
    }
  }
}


