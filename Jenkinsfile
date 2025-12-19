pipeline {
    agent any

    stages {
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

        stage('Deploy') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'f98af94e-037c-4fa1-93ae-3429d5349d57', keyFileVariable: 'private_key', usernameVariable: 'username')]) {
          		    sh 'ssh -i ${private_key} ${username}@docker "\
         		    	docker rm -f examnodejsapp || true && \
         		    	docker run --name examnodejsapp --pull always -p 4444:4444 -d ttl.sh/examnodejsapp:2h \
          		    "'
                }
            }
        }
    }
}
