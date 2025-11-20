pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', defaultValue: '8.2.1', description: 'Enter the application version (e.g., 8.2.1)')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/lakshmiprasad2019/myweb.git'
            }
        }

        stage('Sonar Scan') {
            steps {
                
                sh 'mvn sonar:sonar \
    -Dsonar.projectKey=myweb \
    -Dsonar.host.url=http://54.243.5.208:9000 \
    -Dsonar.login=bd66b6c749fa8e8e383542bf6562e056a7a51547'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Nexus Artifact Upload') {
            steps {
                nexusArtifactUploader(
                    artifacts: [[
                        artifactId: 'myweb',
                        classifier: '',
                        file: "target/myweb-${APP_VERSION}.war",
                        type: 'war'
                    ]],
                    credentialsId: 'nexus3',
                    groupId: 'in.javahome',
                    nexusUrl: '172.31.26.230:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'maven-releases',
                    version: "${APP_VERSION}"
                )
            }
        }

        stage('Tomcat Deploy') {
            steps {
                sh """
                    sudo scp /var/lib/jenkins/workspace/MonolythicFinalProject/target/myweb-${APP_VERSION}.war \
                    root@172.31.18.215:/opt/apache-tomcat-9/webapps/
                    sudo ssh root@172.31.18.215 tomcatstop
                    sudo ssh root@172.31.18.215 tomcatstart
                """
            }
        }
    }
}
