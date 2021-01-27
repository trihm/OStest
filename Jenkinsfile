pipeline {
    agent any

    stages {
        stage('Deploy'){
            steps {
                ansiblePlaybook(credentialsId: 'root-cre-for-ansible', inventory: 'inventory/create_ansible_user_env/', playbook: 'create_ansible_user.yml')
                ansiblePlaybook(vaultCredentialsId: 'ansible-vault-pass', inventory: 'inventory/data_trans_env/', playbook: 'data-trans.yml')
            }
        }
    }
}