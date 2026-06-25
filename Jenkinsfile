pipeline {
    agent any

    parameters {
        choice(name: 'TARGET_ENV',
               choices: ['dev', 'staging', 'prod'],
               description: 'Target Environment')

        choice(name: 'ACTION',
               choices: ['upgrade', 'downgrade'],
               description: 'Action')

        booleanParam(name: 'PATCH_NGINX',
                     defaultValue: false,
                     description: 'Patch nginx?')

        booleanParam(name: 'PATCH_APACHE',
                     defaultValue: false,
                     description: 'Patch apache?')

        booleanParam(name: 'PATCH_OPENSSH',
                     defaultValue: false,
                     description: 'Patch openssh?')

        string(name: 'NGINX_VERSION',
               defaultValue: '1.26.0-1~jammy',
               description: 'nginx version')

        string(name: 'APACHE_VERSION',
               defaultValue: '2.4.52-1ubuntu4.14',
               description: 'apache2 version')

        string(name: 'OPENSSH_VERSION',
               defaultValue: '1:8.9p1-3ubuntu0.15',
               description: 'openssh-server version')

        booleanParam(name: 'DRY_RUN',
                     defaultValue: true,
                     description: 'Dry Run? (no actual changes)')
    }

    stages {
        stage('Pre-Flight Check') {
            steps {
                echo "Environment : ${params.TARGET_ENV}"
                echo "Action      : ${params.ACTION}"
                echo "Dry Run     : ${params.DRY_RUN}"
                echo "nginx       : ${params.PATCH_NGINX} | ${params.NGINX_VERSION}"
                echo "apache      : ${params.PATCH_APACHE} | ${params.APACHE_VERSION}"
                echo "openssh     : ${params.PATCH_OPENSSH} | ${params.OPENSSH_VERSION}"
            }
        }

        stage('Connectivity Check') {
            steps {
                sh """
                ansible -i /home/rahulsharma/project/patch-management/inventory/hosts.ini \
                    ${params.TARGET_ENV} -m ping
                """
            }
        }

        stage('Dry Run') {
            when {
                expression { params.DRY_RUN == true }
            }
            steps {
                sh """
                ansible-playbook \
                    -i /home/rahulsharma/project/patch-management/inventory/hosts.ini \
                    /home/rahulsharma/project/patch-management/playbooks/patch.yml \
                    -e "target_env=${params.TARGET_ENV}" \
                    -e "patch_nginx=${params.PATCH_NGINX}" \
                    -e "patch_apache=${params.PATCH_APACHE}" \
                    -e "patch_openssh=${params.PATCH_OPENSSH}" \
                    -e "nginx_version=${params.NGINX_VERSION}" \
                    -e "apache_version=${params.APACHE_VERSION}" \
                    -e "openssh_version=${params.OPENSSH_VERSION}" \
                    --check --diff
                """
            }
        }

        stage('Approval') {
            when {
                expression { params.TARGET_ENV == 'prod' }
            }
            steps {
                input message: "Production pe patch karna hai?", ok: "Haan, karo!"
            }
        }

        stage('Apply Patch') {
            when {
                expression { params.DRY_RUN == false }
            }
            steps {
                sh """
                ansible-playbook \
                    -i /home/rahulsharma/project/patch-management/inventory/hosts.ini \
                    /home/rahulsharma/project/patch-management/playbooks/patch.yml \
                    -e "target_env=${params.TARGET_ENV}" \
                    -e "patch_nginx=${params.PATCH_NGINX}" \
                    -e "patch_apache=${params.PATCH_APACHE}" \
                    -e "patch_openssh=${params.PATCH_OPENSSH}" \
                    -e "nginx_version=${params.NGINX_VERSION}" \
                    -e "apache_version=${params.APACHE_VERSION}" \
                    -e "openssh_version=${params.OPENSSH_VERSION}"
                """
            }
        }

        stage('Version Verify') {
            when {
                expression { params.DRY_RUN == false }
            }
            steps {
                sh """
                ansible -i /home/rahulsharma/project/patch-management/inventory/hosts.ini \
                    ${params.TARGET_ENV} \
                    -m shell -a 'dpkg -l | grep -E "nginx|apache2|openssh-server"'
                """
            }
        }

        stage('Generate Report') {
            when {
                expression { params.DRY_RUN == false }
            }
            steps {
                sh """
                ansible -i /home/rahulsharma/project/patch-management/inventory/hosts.ini \
                    ${params.TARGET_ENV} \
                    -m fetch -a 'src=/tmp/patch_report.txt dest=/home/rahulsharma/project/patch-management/reports/ flat=no'
                """
                echo "Report saved in reports/ directory"
            }
        }
    }

    post {
        success {
            echo "SUCCESS | ${params.TARGET_ENV} | ${params.ACTION} complete"
        }
        failure {
            echo "FAILED | ${params.TARGET_ENV} | Check logs"
        }
    }
}
