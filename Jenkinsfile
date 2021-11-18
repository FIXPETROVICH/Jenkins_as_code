pipeline {
    agent { label 'test3' }
    triggers { 
        cron('H 2 * * 7')
    }
    environment {
                  network = "192.168.0.105/24"
        
    }
    options {
        buildDiscarder(logRotator(numToKeepStr:'5'))
    }
    stages {
        stage('Clone repository') {
            steps {
                    git url: 'https://github.com/FIXPETROVICH/Jenkins_as_code.git', branch:'master'
            }
        }

        stage('Host scan') {
            steps {
                   sh """
                       sudo apt update -yq
                       sudo apt install nmap -yq
                       if [ ! -d results ]; then mkdir results;fi
                       nmap $network -n -sP | grep report | awk '{print \$5}' > results/goal_${env.BUILD_ID}.txt
                     
                    """
            }
        }

        stage('Internet access speed test') {
            steps {
                   sh """
                       wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
                       chmod +x speedtest-cli
                       echo "#####################################################" >> results/goal_${env.BUILD_ID}.txt
                       ./speedtest-cli >> results/goal_${env.BUILD_ID}.txt
                   """
            }
        }

        stage('Saving results') {
            steps {
                withCredentials([string(credentialsId: 'some_randome', variable: 'token')]) {
                    
                     sh 'git add results/'
                     sh 'git commit -m "Add report_$BUILD_ID"'
                     sh 'git push https://$token@github.com/FIXPETROVICH/Jenkins_as_code.git'
                    
                }
            }
        }

    }
    post {
            always {
                deleteDir()
            }
            success {
                slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
            failure {
                slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
    }
}
