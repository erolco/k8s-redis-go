pipeline {
    agent any
    tools {
        go 'go1.14'
    }
    environment {
        GOCACHE = '/tmp/gocache'
        GO114MODULE = 'on'
        CGO_ENABLED = 0 
        GOPATH = "${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}"
    }
    stages {        
        stage('Pre Test') {
            steps {
                echo 'Installing dependencies'
                sh 'go version'
                sh 'go get -u golang.org/x/lint/golint'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Compiling and building'
                sh 'go build'
            }
        }

        stage('Test') {
            steps {
                withEnv(["PATH+GO=${GOPATH}/bin"]){
                    echo 'Running vetting'
                    sh 'go vet .'
                    echo 'Running linting'
                    sh 'golint .'
                    echo 'Running test'
                    sh 'cd test && go test -v'
                }
            }
        }
        stage('Docker') {
            agent any
            environment {

                DOCKER_CREDENTIALS = credentials('erolco')
            }
            when {
                branch 'master'
            }

            steps {                           
                // Use a scripted pipeline.
                script {
                    node {
                        def app

                        stage('Clone repository') {
                            checkout scm
                        }

                        stage('Build image') {                            
                            app = docker.build("${env.DOCKER_CREDENTIALS_USR}/k8-redis-go")
                        }

                        stage('Push image') {  
                            // Use the Credential ID of the Docker Hub Credentials we added to Jenkins.
                            docker.withRegistry('https://registry.hub.docker.com', 'erolco') {                                
                                // Push image and tag it with our build number for versioning purposes.
                                app.push("${env.BUILD_NUMBER}")                      

                                // Push the same image and tag it as the latest version (appears at the top of our version list).
                                app.push("latest")
                            }
                        }              
                    }                 
                }
            }
        }
    }
        
    }

}
