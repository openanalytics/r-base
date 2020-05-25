pipeline {

    agent {
        kubernetes {
            yamlFile 'kubernetesPod.yaml'
        }
    }

    options {
        authorizationMatrix(['hudson.model.Item.Build:consultants', 'hudson.model.Item.Read:consultants'])
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    environment {
        shortCommit = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
        IMAGE = "r-base"
        NS = "openanalytics"
        REG = "196229073436.dkr.ecr.eu-west-1.amazonaws.com"
    }
    
    stages {

        stage('Pulling Old Image'){
            steps {
                container('dind') {
                    // login to ecr
                    ecrPull "${env.REG}", "${env.NS}/${env.IMAGE}", "latest", '', 'eu-west-1'
                }
            }
        }
        
        stage('Build openanalytics/r-base Docker'){
            steps {
                container('dind'){
                    sh """
                        docker build --cache-from ${env.REG}/${env.NS}/${env.IMAGE}:latest -t ${env.NS}/${env.IMAGE} -t ${env.NS}/${env.IMAGE}:${env.shortCommit} .
                    """

                }
            }
        }
    }

    post {
        success  {
            container('dind'){
                sh "echo tagging and pushing images to registry"

                ecrPush "${env.REG}", "${env.NS}/${env.IMAGE}", "latest", '', 'eu-west-1' 
                ecrPush "${env.REG}", "${env.NS}/${env.IMAGE}", "${env.shortCommit}", '', 'eu-west-1'   
            }
        }
    }
}