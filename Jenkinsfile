pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                // Достаем секретный файл (ключ) из Jenkins
                withCredentials([file(credentialsId: 'ansible-ssh-key-file', variable: 'SSH_KEY_PATH')]) {
                    script {
                        echo "Запускаем Ansible..."
                        
                        // 1. Копируем файл ключа из секретного хранилища в рабочую папку workspace
                        // чтобы Docker мог его увидеть при монтировании
                        bat 'copy "%SSH_KEY_PATH%" ansible_key_temp'
                        
                        // 2. Запускаем Docker с Ansible
                        // Мы монтируем текущую папку (%WORKSPACE%) внутрь контейнера в папку /work
                        bat """
                            docker run --rm ^
                            -v "%WORKSPACE%":/work ^
                            willhallonline/ansible:alpine ^
                            sh -c "cp /work/ansible_key_temp /tmp/ssh_key && chmod 600 /tmp/ssh_key && ansible-playbook -i /work/ansible/inventory.ini --private-key /tmp/ssh_key /work/ansible/playbook.yml"
                        """
                        
                        // 3. Удаляем временный ключ с диска Windows после работы (для безопасности)
                        bat 'del ansible_key_temp'
                    }
                }
            }
        }
    }
}
