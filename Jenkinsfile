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
        VERSION = '4.0.2'
        DOCKER_BUILDKIT = '1'
    }
    
    stages {

        stage('Build openanalytics/r-base Docker'){
            steps {
                container('dind'){
                    withOARegistry {
                
                       sh """
                           docker build --build-arg BUILDKIT_INLINE_CACHE=1 \
                             --cache-from ${env.REG}/${env.NS}/${env.IMAGE}:latest \
                             -t ${env.NS}/${env.IMAGE}:${env.VERSION} \
                             -t ${env.NS}/${env.IMAGE}:${env.shortCommit} \
                             -t ${env.NS}/${env.IMAGE}:latest \
                             .
                       """
                   }
                }
            }
        }
    }

    post {
        success  {
            container('dind'){
                sh "echo tagging and pushing images to OA registry and Docker Hub"
                
                withOARegistry {

                    ecrPush "${env.REG}", "${env.NS}/${env.IMAGE}", "latest", '', 'eu-west-1' 
                    ecrPush "${env.REG}", "${env.NS}/${env.IMAGE}", "${env.VERSION}", '', 'eu-west-1'
                    ecrPush "${env.REG}", "${env.NS}/${env.IMAGE}", "${env.shortCommit}", '', 'eu-west-1'
                
                }
                
                withDockerRegistry([
                                    credentialsId: "openanalytics-dockerhub",
                                    url: ""]) {
                                
                                sh "docker push ${env.NS}/${env.IMAGE}:${env.VERSION}"
                                sh "docker push ${env.NS}/${env.IMAGE}:${env.shortCommit}"
                                sh "docker push ${env.NS}/${env.IMAGE}:latest"
                            }
                   
            }
        }
    }
}