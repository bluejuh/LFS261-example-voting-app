pipeline {
  agent none

  stages{

    stage("worker-build"){
      when{
        changeset "**/worker/**"
      }
      agent{
        docker{
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps{
        echo 'Compiling worker app'
        dir('worker'){
          sh 'mvn compile'
        }
      }
    }

    stage("worker-test"){
      when{
        changeset "**/worker/**"
      }
      agent{
        docker{
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps{
        echo 'Running Unit Tests on worker app'
        dir('worker'){
          sh 'mvn clean test'
        }
      }
    }

    stage("worker-package"){
      when{
        branch 'master'
        changeset "**/worker/**"
      }
      agent{
        docker{
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps{
        echo 'Packaging worker app'
        dir('worker'){
          sh 'mvn package -DskipTests'
          archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
      }
    }

    stage('worker-docker-package'){
      agent any
      when{
        changeset "**/worker/**"
        branch 'master'
      }
      steps{
        echo 'Packaging worker app with docker'
        script{
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def workerImage = docker.build("facefrontletsgo/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("latest")
          }
        }
      }
    }

    stage("result-build"){
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }
      }
      when{
        changeset "**/result/**"
      }
      steps{
        echo 'building Result NodeJS Application'
        dir('result'){
          sh 'npm install'
        }
      }
    }

    stage("result-test"){
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }
      }
      when{
        changeset "**/result/**"
      }
      steps{
        echo 'Running Tests on Result NodeJS Application'
        dir('result'){
          sh 'npm install'
          sh 'npm test'
        }
      }
    }

    stage('result-docker-package') {
      agent any
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'Packaging result app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def resultImage = docker.build("facefrontletsgo/result:v${env.BUILD_ID}", './result')
            resultImage.push()
            resultImage.push("${env.BRANCH_NAME}")
            resultImage.push('latest')
          }
        }
      }
    }

    stage('vote-build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
          }
        }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Compiling vote app.'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }
      }
    }

    stage('vote-test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
          }
      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Tests on vote app.'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }
      }
    }

    stage('vote integration'){ 
      agent any 
      when{ 
        changeset "**/vote/**" 
        branch 'master' 
      } 
      steps{ 
        echo 'Running Integration Tests on vote app' 
        dir('vote'){ 
          sh 'sh integration_test.sh' 
        } 
      } 
    } 

    stage('vote-docker-package') {
      agent any
      when{ 
        changeset "**/vote/**" 
        branch 'master' 
      } 
      steps {
        echo 'Packaging vote app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
            def voteImage = docker.build("facefrontletsgo/vote:${env.GIT_COMMIT}", "./vote")
            voteImage.push()
            voteImage.push("${env.BRANCH_NAME}")
            voteImage.push("latest")
          }
        }
      }
    }

    stage('deploy to dev'){
      agent any
      when{
        branch 'master'
      }
      steps{
        echo 'Deploy instavote app with docker compose'
        sh 'docker-compose up -d'
      }
    }

  }

  post{
    always{
      echo 'Building multibranch monopipeline for instavote is completed..'
    }
    failure{
      slackSend (channel: "#ci-cd", message: "Build Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
    }
    success{
      slackSend (channel: "#ci-cd", message: "Build Success: ${env.JOB_NAME} ${env.BUILD_NUMBER}")
    }
  }
}
