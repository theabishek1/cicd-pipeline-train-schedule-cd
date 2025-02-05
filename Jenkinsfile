pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                sshPublisher(
                    continueOnError: false,
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'staging',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'dist/trainSchedule.zip',
                                    removePrefix: 'dist/',
                                    remoteDirectory: '/tmp',
                                    execCommand: '''
                                        sudo systemctl stop train-schedule
                                        sudo rm -rf /opt/train-schedule/*
                                        sudo unzip /tmp/trainSchedule.zip -d /opt/train-schedule
                                        sudo systemctl start train-schedule
                                    '''
                                )
                            ],
                            usePromotionTimestamp: false,
                            verbose: true
                        )
                    ]
                )
            }
        }
    }
}

