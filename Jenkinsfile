pipeline {
    agent {label "agentfarm" }
    environment {
        KEY_FILE = '/home/ubuntu/.ssh/vm-instance-key.pem'
        USER = 'ubuntu'
    }

    stages {
        stage('Delete the workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Installing Ansible') {
           steps {
             script {
               def ansible_exists = fileExists '/usr/bin/ansible'
               if (ansible_exists == true) {
                    echo "Skipping Ansible install - already exists"              
              } else {
              sh 'sudo apt-get update -y && sudo apt-get upgrade -y'
              sh 'sudo apt install -y wget tree unzip ansible python3-pip python3-apt'
            }
         }
       }
    }
        stage('Download Ansible Code') {
            steps {
                git credentialsId: 'git-repo-creds', url: 'git@github.com:sukhdeepsinghh/ansible-webserver1.git'
            }
        }
        stage('Run ansible-lint against playbook') {
            steps {
            sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 apache-install.yml'
            sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 website-update.yml'
            sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint:4 website-test.yml'
            }
       }
        stage('Install apache & update website') {
        steps {
        sh 'export ANSIBLE_HOST_KEY_CHECKING=False && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/apache-install.yml'
        sh 'export ANSIBLE_ROLES_PATH=/opt/jenkins/workspace/ansible-pipeline/roles && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-update.yml'
       }
       }
       stage('Test Website')
       steps {
       sh 'export ANSIBLE_ROLES_PATH=/opt/jenkins/workspace/ansible-pipeline/roles && ansible-playbook -u $USER --private-key $KEY_FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-test.yml'
       }
       }

       stage('Send Slack Notification'){
            steps {
                slackSend color: 'warning', message: "Mr. Deeds: Please approve ${env.JOB_NAME} $env.BUILD_NUMBER} (<${env.JOB_URL} | Open>)"
            }
        }
        stage('Request Input') {
            steps {
               input 'Please approve or deny this build'
            }
        }
   }
   post {
		success {
			 slackSend color: 'warning', message: "Build ${env.JOB_NAME} ${env.BUILD_NUMBER} was successful! :)"
		}
		failure {
			slackSend color: 'warning', message: "Build ${env.JOB_NAME} ${env.BUILD_NUMBER} failed :)"
		}
	}
}
