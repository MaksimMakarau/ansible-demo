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
                // Мы запрашиваем ДВА секрета: 
                // 1. Файл ключа SSH (file)
                // 2. Пароль для sudo (string/text)
                withCredentials([
                    file(credentialsId: 'ansible-ssh-key-file', variable: 'SSH_KEY_PATH'),
                    string(credentialsId: 'debian-sudo-pass', variable: 'SUDO_PASSWORD')
                ]) {
                    script {
                        echo "Запускаем Ansible..."
                        
                        // 1. Копируем файл ключа из секретного хранилища в рабочую папку workspace
                        // чтобы Docker мог его увидеть при монтировании
                        bat 'copy "%SSH_KEY_PATH%" ansible_key_temp'
                        
                        // 2. Запускаем Docker с Ansible
                        // Мы монтируем текущую папку (%WORKSPACE%) внутрь контейнера в папку /work
                        // Мы добавляем строку: -e ANSIBLE_BECOME_PASS=%SUDO_PASSWORD%
                        // Jenkins подставит значение секрета в переменную SUDO_PASSWORD
                        // А Docker передаст её внутрь контейнера как ANSIBLE_BECOME_PASS
                        bat """
                            docker run --rm ^
                            -v "%WORKSPACE%":/work ^
                            -e ANSIBLE_HOST_KEY_CHECKING=False ^
                            -e ANSIBLE_BECOME_PASS=%SUDO_PASSWORD% ^
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
