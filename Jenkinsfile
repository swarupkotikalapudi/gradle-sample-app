pipeline {
    agent {
        node {
            label 'linuxagent'
        }
    }
    tools {
        jdk 'JDK_1.8'
    }
    environment {
        DockerImageName = "swarupkotikalapudi/gradle-example"
    }

    // trigger at everyday at midnight
    //triggers {
    //    pollSCM('0 0 * * *')
    //}

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()                  
            }
        }
        stage('checkout stage'){
            environment {
                GitHubCred = credentials('GitHubCredentials')
            }
            steps{
                //checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/swarupkotikalapudi/gradle-sample-app.git']])
                // picks change in any branch due to github webhook
                checkout scmGit(branches: [[name: '*/*']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/swarupkotikalapudi/gradle-sample-app.git']])
            }
        }
        stage('Clean') {
            steps {
                gradlew('clean')
            }
        }
        stage('Compile') {
            steps {
                gradlew('build')
            }
        }
        stage('Publish') {
            steps {
                gradlew('publish')
            }
        }
        stage('Build Docker Image') {
            steps {
                gradlew('jibdockerbuild')
            }
        }
        stage('Login to Docker Hub') {
            environment {
                dockerhub_cred = credentials('DockerhubCred')
            }
            steps {
	            sh 'echo $dockerhub_cred_PSW | docker login -u $dockerhub_cred_USR --password-stdin'
            }
        }
        stage('Push to docker hub') {
            environment {
                DockerImageName = "swarupkotikalapudi/gradle-example"
            }
            steps {
                sh 'docker push $DockerImageName'
            }
        }
    }
    post {
        always {
            //sh "docker rmi $DockerImageName"
            sh "docker image prune -af"
            sh 'docker logout'
        }
        success {
            slackSend(
                channel: '#jenkinintegration',
                color: 'good',
                message: "Build successful: ${currentBuild.fullDisplayName}"
            )
        }
        failure {
            slackSend(
                channel: '#jenkinintegration',
                color: 'danger',
                message: "Build failed: ${currentBuild.fullDisplayName}"
            )
        }
    }
}

def gradlew(String... args) {
    sh "./gradlew ${args.join(' ')} -s"
}
