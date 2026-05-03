pipeline {
    agent {
            label 'AGENT-1'
         }
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 30, unit: 'MINUTES')
       //  disableConcurrentBuilds()
         ansiColor('xterm')
    }
    //  parameters {
    //    choice(name: 'action', choices: ['Apply', 'Destroy'], description: 'Pick something')
    //  }
    environment {
        def appVersion = '' //variable declaration here.
        nexusURl = 'nexus.lithesh.shop:8081'

 
    }
    stages {
        stage('read the version') {
            steps {
                script {
                def packageJson = readJSON file: 'package.json'
                appVersion = packageJson.version
                echo "application version: $appVersion"
            }
            }
        }
        stage('NPM Dependencies') { 
            steps {
                sh """
                  npm install
                  ls -ltr
                  echo "application version: $appVersion"
                  
                """ 
            }
        }
         stage('Build') { 
            steps {
                sh """
                  zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                  ls -ltr
                  
                """ 
            }
        } 
         stage('Sonar Scan') { 
            environment {
                scannerHome = tool 'sonar-6.0' //refering scanner CLI
            }
            steps {
                script {
                    withSonarQubeEnv('sonar-6.0'){ // referring sonar server
                       sh  "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
         stage("Quality Gate"){
            steps {
                timeout(time:30, unit: 'MINUTES'){
                     waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Nexus Artifact Uploader') { 
            steps {
                script {

                    nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: "${nexusUrl}",
                            groupId: 'com.expense',
                            version: "${appVersion}",
                            repository: "backend",
                            credentialsId: 'nexus-auth',
                            artifacts: [
                                [artifactId: "backend",
                                classifier: '',
                                file: "backend-" + "${appVersion}" + '.zip',
                                type: 'zip']
                            ]
                        )

                }
            }
        }
         stage('Deploy') { 
             steps {
                script {
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'backend-deploy', parameters: params, wait: false
                }   
            }
         }

    }
     post { 
        always { 
            deleteDir()
            echo 'I will always say Hello Always!'
        }
        success { 
            echo 'I will always say Hello Success!'
        }
        failure{ 
            echo 'I will always say Hello failure!'
        } 
    }
}