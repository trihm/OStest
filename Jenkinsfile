pipeline {
    agent any

    stages {
        stage('Deploy'){
            steps {
                ansiblePlaybook(credentialsId: 'root-cre-for-ansible', inventory: 'iac-transfer-server/inventory/create_ansible_user_env/', playbook: 'iac-transfer-server/create_ansible_user.yml')
                ansiblePlaybook(vaultCredentialsId: 'ansible-vault-pass', inventory: 'iac-transfer-server/inventory/data_trans_env/', playbook: 'iac-transfer-server/data-trans.yml')
            }
        }
    }
}