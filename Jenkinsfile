pipeline {
    agent any
    stages {
        stage ('Checkout') {
            options {
                // Don't checkout the repo where it is this Jenkinsfles, just the target repos of this pipeline. Must be set at every stage.
                skipDefaultCheckout true
            }
            parallel {
                stage('Checkout vdc-logging') {
                    options { skipDefaultCheckout true }
                    steps {
                        dir('vdc-logging') {
                            git changelog: false, credentialsId: 'Aitor-IDEKO-GitHub', poll: false, url: 'https://github.com/DITAS-Project/VDC-Logging-Agent.git'
                        }
                    }
                }
                stage('Checkout vdc-request') {
                    options { skipDefaultCheckout true }
                    steps {
                        dir('vdc-request') {
                            git changelog: false, credentialsId: 'Aitor-IDEKO-GitHub', poll: false, url: 'https://github.com/DITAS-Project/VDC-Request-Monitor.git'
                        }
                    }
                }
                stage('Checkout vdc-throughput') {
                    options { skipDefaultCheckout true }
                    steps {
                        dir('vdc-throughput') {
                            git changelog: false, credentialsId: 'Aitor-IDEKO-GitHub', poll: false, url: 'https://github.com/DITAS-Project/VDC-Throughput-Agent.git'
                        }
                    }
                }
            }
        }
        stage ('Build - Test') {
            options {
                skipDefaultCheckout true
            }
            parallel {
                stage('Build - test vdc-logging') {
                    agent {    
                        dockerfile {
                            dir 'vdc-logging'
		                    filename 'Dockerfile.testing' // Dockerfile only at this moment but should be Dockerfile.build
		                    reuseNode true           
			            }
                    }
                    steps {
                        sh "cd /go/src/github.com/DITAS-Project/VDC-Logging-Agent && go test ./..."
                        
                    }
                }
                 stage('Build vdc-throughput') {
                     agent {
                         dockerfile{
                            dir 'vdc-throughput'
                            filename 'Dockerfile.testing'
                            reuseNode true 
                         }
                     }
                     steps {
                        sh "cd ./vdc-throughput/ && mvn test"
                     }
                 }
                // we should test this but jenkins dosen´t let me so ... no testing it is!
                // stage('Test vdc-request-monitor') {
				// 	// We don't need this to be an agent. For this component we just need the image to be generated as there are not Unit Tests
				// 	agent {
                //         dockerfile {
                //             dir 'vdc-request'
                //             filename 'Dockerfile.testing'
                //             reuseNode true 
                //             args '--user 0'
			    //         }	
                //     }
                //     steps {
                //         echo "VDC-Request-Monitor Testing"
                //         sh "export VDC_PORT=80"
                //         sh "export VDC_ADDRESS=google.de"
                //         sh "/run.sh &" //start the server
                //         sh "sleep 25" //wait 
                //         sh "curl -k 'http://127.0.0.1:8000'" //this is the quick test
                //     }
                // }
           }
        }
        stage ('Build - VDC-Request-Monitor Testing') {
            agent none
		steps {
                dir('vdc-request') {
                    echo "Generation the VDC Request image"
                    sh "docker build -t \"ditas/vdc-request-monitor\" ."
                    //TODO: push this image?
                }
                
            }
        }
		// At this point the 3 images were created started and stopped, but they exist (docker images)
		// We can create the final artifact, an image with the best of the three		
        stage ('Main image generation') {
            // The final image must be built at the node itsefl, not inside a container
            agent none
            steps {
               // The Dockerfile.artifact copies the code into the image and run the jar generation.
               echo 'Creating the image...'

               // This will search for a Dockerfile.artifact in the working directory and build the image to the local repository
               sh "docker build --no-cache -t \"ditas/vdc-base-image\" -f Dockerfile.artifact ."
               echo "Done"

               echo 'Retrieving Docker Hub password from /opt/ditas-docker-hub.passwd...'
               // Get the password from a file. This reads the file from the host, not the container. Slaves already have the password in there.
               script {
                   password = readFile '/opt/ditas-docker-hub.passwd'
               }
               echo "Done"

               echo 'Login to Docker Hub as ditasgeneric...'
               sh "docker login -u ditasgeneric -p ${password}"
               echo "Done"

               echo "Pushing the image ditas/vdc-base-image:latest..."
               sh "docker push ditas/vdc-base-image:latest"
               echo "Done "
           }
        }
        // // Deploy the image to the staging seerver
        // stage('Image deploy') {
        //     agent any
        //     options {
        //         // Don't need to checkout Git again
        //         skipDefaultCheckout true
        //     }
        //     steps {
        //         // Deploy to Staging environment calling the deployment script
        //         //  TODO !!!! Edit this file to set the correct iamge tag and port mapping
        //         sh './jenkins/deploy-staging.sh'
        //     }
        // }
        // // Simple test(s) to ensure the base image and all the components are running after deployment
        // stage('Component level testing') {
        //     agent any
        //     options {
        //         // Don't need to checkout Git again
        //         skipDefaultCheckout true
        //     }
        //     steps {
		// 		// TODO any vdc-logging test here? A call to the an API method?
				
				
		// 		// TODO any vdc-logging test here? A call to the an API method?
				
		// 		// vdc-request-monitor test - 31.171.247.162 = Staging environment
		// 		// TODO 8000? It is not mapped in the deploy script, in the docker run command
        //         sh "exec curl -k 'http://31.171.247.162:8000'" // TODO AITOR - deploy an email server to send email on failures
        //     }
        // }
    }
}
