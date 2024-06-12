pipeline {
    agent {
        node {
            label 'linuxagent'
        }
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
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/swarupkotikalapudi/gradle-sample-app.git']])
                checkout([$class: 'GitSCM',
                branches: [[name: '*/master' ]],
                extensions: [[
                    $class: 'CloneOption',
                    shallow: true,
                    depth:   1,
                    timeout: 30
                ]]
                userRemoteConfigs: [[
                    url: 'https://github.com/swarupkotikalapudi/gradle-sample-app.git',
                    credentialsId: 'GitHubCred'
                ]]
            ])
            }
        }
        stage('Compile') {
            steps {
                gradlew('clean', 'build')
            }
        }
        //stage('Publish to Repo') {
        //    steps {
        //        gradlew('publish')
        //    }
        //}
        stage('Build Docker Image') {
            steps {
                gradlew('jibdockerbuild')
            }
        }
        stage('Login to Docker Hub') {
            environment {
                DOCKERHUB_CREDENTIALS_USR = "swarupkotikalapudi"
                DOCKERHUB_CREDENTIALS = credentials('DockerHubCredentials')
            }
            steps {
	        sh 'echo $DOCKERHUB_CREDENTIALS | sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
	        echo 'Login Completed'
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
        stage('Remove Unused docker image') {
            steps{
                sh "docker rmi $DockerImageName"
                sh "docker rmi $DockerImageName:latest"
            }
        }
    }
    post {
        always {
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
