// This adds install and test stages before static code analysis
pipeline {

    environment {
        registry = "change"
        registryCredentials = "dockerhub_id_maybe_change"
        dockerImage = ""
    }

  agent any

  stages {
    stage('Checkout') {
        steps {
          // Get some code from a GitHub repository
          git branch: 'main', url: 'https://github.com/nuddsy/lbg-car-build-app-starter.git'
        }
    }
    stage('Checkout additional repo') {
        steps {
          // Get some code from a GitHub repository
          dir('front-end-repo') {
            git branch: 'main', url: 'https://github.com/nuddsy/lbg-car-react-starter'
          }

          dir('back-end-repo') {
            git branch: 'main', url: 'https://github.com/nuddsy/lbg-car-spring-app-starter'
          }
        }
    }
    stage('Copy Repos') {
        steps {
          // Get some code from a GitHub repository
          sh 'cp -r front-end-repo/* lbg-car-build-app-starter/'
          sh 'cp -r back-end-repo/* lbg-car-build-app-starter/'
          sh 'cd lbg-car-build-app-starter'
        }
    }
    stage('Compile back end') {
        steps {
            //Run Maven on Unix agent
            sh "mvn clean compile"
        }
    }
    stage('Test back end') {        
        steps { sh 'mvn -D.maven.compile.skip test'}         
    }
    // maybe delete
    // stage('Package back end') {           
    //    steps { sh 'mvn -D.maven.test.skip -D.maven.compile.skip package'}       
    //}
    stage('Install front end') {
        steps {
            // Install the ReactJS dependencies
            sh "npm install"
        }
    }
    stage('Test front end') {
        steps {
          // Run the ReactJS tests
          sh "npm test"
        }
    }
    // maybe delete
    stage('Build front end') {
        steps {
          // Run the ReactJS tests
          sh "npm run build"
        }
    }
    stage ('Build Docker Images'){
        steps{
            script {
                sh 'docker-compose build'
            }
        }
    }
    // check with Adam
    stage ("Push to Docker Hub"){
        steps {
            script {
                sh 'docker-compose push'
            }
        }
    }
    // check with Adam
    // stage ("Deploy to Server")

    // ask if Jenkins cleans up 
    stage ("Clean up"){
        steps {
            script {
                sh 'docker image prune --all --force --filter "until=48h"'
            }
        }
    }
}
}