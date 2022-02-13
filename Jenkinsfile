pipeline {

    agent none

    stages {
        stage('worker build') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('worker test') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Running Unit tests on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker package') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker docker-package') {
            agent any
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'credentials-id') {
                        def workerImage = docker.build("aniruth/worker:v${env.BUILD_ID}", "./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('vote Build') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Compiling app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        stage('vote Test') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset "**/vote/**"
            }
            steps {
                echo 'Running Unit tests'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }
        stage('vote docker-package') {
            agent any
            when {
                branch 'master'
                changeset "**/vote/**"
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'credentials-id') {
                        def voteImage = docker.build("aniruth/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
        stage('result Build') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Compiling app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage('result Test') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Running Unit tests'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('result docker-package') {
            agent any
            when {
                branch 'master'
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging result app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'credentials-id') {
                        def resultImage = docker.build("aniruth/result:v${env.BUILD_ID}", "./result")
                        resultImage.push()
                        resultImage.push("${env.BRANCH_NAME}")
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed successfully'
        }
    }
}
