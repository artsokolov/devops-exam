pipeline {
    agent any

    stages {
        stage('Run tests') {
            steps {
                sh '''
                    docker run --rm \
                      -v "$PWD:/app" \
                      -w /app \
                      node:24-alpine \
                      sh -c "npm ci && npm test"
                '''
            }
        }

        stage('Add known host') {
            steps {
                sh "mkdir -p ~/.ssh"
                sh "ssh-keyscan -H docker >> ~/.ssh/known_hosts"
            }
        }

    	stage('Build docker image') {
    	    steps {
    	    	sh "docker build . --tag ttl.sh/examnodejsapp:2h"
    		    sh "docker push ttl.sh/examnodejsapp:2h"
    	    }
    	}

        stage('Deploy to target machine') {
            steps {
                sshagent(credentials: ['f98af94e-037c-4fa1-93ae-3429d5349d57']) {
                    sh '''
                        ansible-playbook -i inventory.ini ansible/deploy.yml
                    '''
                }
            }
        }

        stage('Deploy to docker machine') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'f98af94e-037c-4fa1-93ae-3429d5349d57', keyFileVariable: 'private_key', usernameVariable: 'username')]) {
          		    sh 'ssh -i ${private_key} ${username}@docker "\
         		    	docker rm -f examnodejsapp || true && \
         		    	docker run --name examnodejsapp --pull always -p 4444:4444 -d ttl.sh/examnodejsapp:2h \
          		    "'
                }
            }
        }

        stage('Deploy to k8s') {
            steps {
    	        withKubeConfig([credentialsId: 'jenkins-kube-token', serverUrl: 'https://kubernetes:6443']) {
    			    sh 'kubectl apply -f pod.yaml'
    		    }
            }
        }
    }
}
