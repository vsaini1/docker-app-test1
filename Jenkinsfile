pipeline{
    agent any
    tools {
      maven 'maven3'
    }
    environment {
      DOCKER_TAG = getVersion()
    }
    stages{
        stage('GitHub Webhook'){
            steps{
                git branch: 'main', credentialsId: 'git-creds1', 
                    url: 'https://github.com/vsaini1/docker-app-test1'
            }
        }
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        stage('Docker Build'){
            steps{
                sh "docker build . -t vsainiibm/docappt1:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dhup1', passwordVariable: 'dhpwd1', usernameVariable: 'dhuser1')]) {
                   sh "docker login -u ${dhuser1} -p ${dhpwd1} "
                    }
                sh "docker push vsainiibm/docappt1:${DOCKER_TAG} "
            }
        }
        stage('Ansible Setup Docker'){
            steps{
                //sh "echo ‘[dev] >dev.inv; curl ifconfig.me >>dev.inv"
                ansiblePlaybook credentialsId: 'dev-cred1', disableHostKeyChecking: true, installation: 'ansible', inventory: 'dev.inv', playbook: 'setup-docker.yml'
                //ansiblePlaybook credentialsId: 'srv-cred1', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'setup-docker.yml'
            }
        }
        //stage('Delete Old Container'){
          //  steps{
 	        //sshagent (['dev-cred1']) {
	         //sh "ssh -o StrictHostKeyChecking=no -t azureuser@10.0.0.5  echo docker_hulu_was_not_running "
			 //sh "ssh -o StrictHostKeyChecking=no azureuser@10.0.0.5  docker rm hulu "
	           // }
            //}
	    //}
        
        stage('Docker run'){
            steps{
                sshagent(['dev-cred1']) {
                //def dockerScript = "docker run -d --name=hulu -p 8282:8080 vsainiibm/docappt1:${DOCKER_TAG}"
                sh "ssh -o StrictHostKeyChecking=no -tt azureuser@10.0.0.5 'docker stop hulu;docker rm hulu;docker run -d --name=hulu -p 8282:8080 vsainiibm/docappt1:${DOCKER_TAG}'"
                }
            }
        }
    }    
}
def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
