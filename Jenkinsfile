#!/usr/bin/groovy

//variables
def REPO_URL = "https://github.com/YevhenVieskov/drupal-nginx.git"
def FOLDER_NAME = "/home/ubuntu/drupal-nginx/"
def DEST_FOLDER_NAME = "/home/ubuntu/"

properties([pipelineTriggers([githubPush()])])

pipeline {

    parameters {    
        string(name: 'IP_DEPLOY', defaultValue: '52.14.158.47', description: 'change the IP address of Drupal server')        
    }

    agent any

    options {
        disableConcurrentBuilds()
    }	

    stages {

		//Download code from the repository
		stage("Checkout") {			 
            steps {
                git credentialsId: 'github-ssh-key', url: "${REPO_URL}", branch: 'main' 
            }
	    } 

        stage("Clone repository to swarm manager") {			 
            steps {
                sshagent(['ssh-swarm']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${params.IP_DEPLOY} git clone ${REPO_URL} || true"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${params.IP_DEPLOY} cp ${FOLDER_NAME}docker-stack.yml ${DEST_FOLDER_NAME}"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${params.IP_DEPLOY} rm -rf ${FOLDER_NAME}"				
                }         
            }
	    }

        stage("Deploy docker stack to docker swarm") {			 
            steps {
                sshagent(['ssh-swarm']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${params.IP_DEPLOY} docker stack deploy --compose-file=${DEST_FOLDER_NAME}docker-stack.yml drupal"				
                }         
            }
	    }

        stage("Approve") {
            steps { approve('Do you want to destroy your application?') }
		}

        stage("Delete application and clean up server") {			 
            steps {
                sshagent(['ssh-swarm']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${params.IP_DEPLOY} docker stack rm drupal"					
                }         
            }
	    }
    }
}


def approve(msg) {

	timeout(time:1, unit:'DAYS') {
		input(msg)     //'Do you want to deploy to production?'
	}
}

// https://dzone.com/articles/deploy-docker-compose-services-swarm
// https://docs.docker.com/engine/reference/commandline/stack_deploy/
