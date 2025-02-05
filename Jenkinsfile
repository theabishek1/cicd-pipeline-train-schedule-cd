pipeline {
    agent any

    environment {
        JAVA_HOME = 'C:\\cicd\\jdk\\openjdk-21'
        PATH = "${env.JAVA_HOME}\\bin;${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from your GitHub repository
                git credentialsId: 'github-token', url: 'https://github.com/theabishek1/cicd-pipeline-train-schedule-cd.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                echo 'Running build automation with Gradle'
                // For Windows, using 'bat' instead of 'sh'
                bat 'gradlew.bat build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: '''
                                            sudo systemctl stop train-schedule &&
                                            rm -rf /opt/train-schedule/* &&
                                            unzip /tmp/trainSchedule.zip -d /opt/train-schedule &&
                                            sudo systemctl start train-schedule
                                        '''
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                input 'Are you sure you want to deploy to production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: '''
                                            sudo systemctl stop train-schedule &&
                                            rm -rf /opt/train-schedule/* &&
                                            unzip /tmp/trainSchedule.zip -d /opt/train-schedule &&
                                            sudo systemctl start train-schedule
                                        '''
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
