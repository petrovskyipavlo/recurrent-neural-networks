
properties([pipelineTriggers([githubPush()])])

pipeline {
    agent any         

    options {
        disableConcurrentBuilds()
    }

	environment {		
		registry1 = "vieskov1980/intermine_builder" 
        registryCredential = 'docker_hub_login' 
        //dockerImage = '' 

	}

    stages {

		//Download code from the repository
		stage("Checkout") {			 
            steps {
                               
                git credentialsId: 'github-key', url: 'https://github.com/petrovskyipavlo/final_task_iac.git', branch: 'main'
            }
	    }

        

        stage('Build image with app') {
            steps{
                sh  "docker-compose up --build flask"
                
            }
        } 
        
        stage('Push app image to dockerhub ') {
            steps{ 
                script{
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        sh "docker-compose push"
                    }
                }
           }
        }

               


        stage('Deploy containers to Server') {
            steps{
                sshagent (credentials: ['stage']) {
                    sh 'ssh -o StrictHostKeyChecking=no  pavelp@192.168.0.105 "git clone https://github.com/petrovskyipavlo/recurrent-neural-networks.git && cd recurrent-neural-networks  docker-compose -f dockerhub.docker-compose.yml --build flask"'
                } 
            }
        }

               
        

        stage("Approve Delete Container on Dev Environment") {
            steps { approve('Do you want to delete application to production?') }
		}

        stage("Delete Container on Dev Environment") {
            steps {
                sshagent (credentials: ['stage']) {               
                    sh 'ssh -o StrictHostKeyChecking=no  pavelp@192.168.0.105 "cd recurrent-neural-networks && docker-compose -f dockerhub.docker-compose.yml down && docker system prune -a -f"'
                }
            }
        }


               

    } 
} 

def approve(msg) {
	timeout(time:1, unit:'DAYS') {
		input(msg)     
	}
}