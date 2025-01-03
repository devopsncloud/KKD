pipeline {
    agent any

    environment {
        SERVER_IP = credentials('deploy_server_id')
    }
    stages {
        stage('Setup') {
            steps {
		sh '''
		python3 -m venv venv  # Create virtual environment
        	source venv/bin/activate  # Activate virtual environment
		pip install --upgrade pip  # Upgrade pip to the latest version
                pip install -r requirements.txt
			
		'''
            }
        }
        stage('Test') {
            steps {
		sh '''
        	source venv/bin/activate  # Activate the virtual environment again if needed
        	pytest  # Run the tests with pytest
       		 '''
            }
        }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
                sh "ls -lart"
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy_server_sshkey', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip  ${username}@${SERVER_IP}:/home/ec2-user/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
                        source app/venv/bin/activate
                        cd /home/ec2-user/app/
                        pip install -r requirements.txt
                        sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
       
        
       
        
    }
}
