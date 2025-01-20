def get_server_properties (user) {
	if ( user == "abc" ) {
		PASSWD="sbcd"
    EMAIL="email.com"
		}
	else if ( user == "def" ) {
		PASSWD="defg"
    EMAIL="mail.com"
		}
	return [ PASSWD, EMAIL ]
	}
	
node {
    currentBuild.displayName = "#${currentBuild.number} ${SERVER_NAME}"
}

pipeline {
    agent {
        label "LABEL"
    }
    parameters {
        string(name: 'SERVER_NAME', defaultValue: '', description: 'Name of the server. eg: <<SERVER>>')
        string(name: 'PYTHON_INTERPRETER', defaultValue: '', description: 'Python used in the server. eg: /usr/bin/python')
        string(name: 'SERVER_USER', defaultValue: '', description: 'User of the server. eg: <<USERNAME>>')
    }
    stages {
        stage ("Scripts_Checkout") {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'ssh://<<URL>>/FamidhaThurab/Ansible/']]])
            }
        }
        stage ("HostFile_Creation") {
            steps {
                script {
                    ( password, mail_id ) = get_server_properties (SERVER_USER)
                    env.PASSWD = password
                    env.EMAIL = mail_id
                    sh 'echo "${SERVER_NAME} ansible_python_interpreter=${PYTHON_INTERPRETER} ansible_user=${SERVER_USER} ansible_password=${PASSWD} ansible_become_password=${PASSWD} gerrit_email=${EMAIL}" > ${WORKSPACE}/hosts'
                }
            }
        }
        stage ("SSH_Key_Transfer") {
            steps {
                script {
                    ( PASSWD, EMAIL ) = get_server_properties (SERVER_USER)
                    env.PASSWD = password
                    sh 'echo "${PASSWD}" > ${WORKSPACE}/file.txt'
                    sh 'sshpass -f ${WORKSPACE}/file.txt ssh-copy-id -i ~/.ssh/id_rsa.pub ${SERVER_USER}@${SERVER_NAME}'
                    sh 'rm -rf ${WORKSPACE}/file.txt'
                }
            }
        }
        stage ("Playbook_Execution") {
            steps {
                sh "ansible-playbook -i ${WORKSPACE}/hosts -e 'workspace=${WORKSPACE}' ${WORKSPACE}/configuration_management/playbook.yml"
            }
        }
    }
}
