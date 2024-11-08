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
          sh 'mkdir -p /var/lib/jenkins/lbg-car-build-app-starter/lbg-car-spring-app-starter' 
          sh 'mkdir -p /var/lib/jenkins/lbg-car-build-app-starter/lbg-car-react-app-starter' // Copy contents from additional repos to main repo 
          sh 'cp -r front-end-repo/* /var/lib/jenkins/lbg-car-build-app-starter/lbg-car-react-app-starter/' 
          sh 'cp -r back-end-repo/* /var/lib/jenkins/lbg-car-build-app-starter/lbg-car-spring-app-starter/' // Ensure we are in the correct directory 
          sh 'cd /var/lib/jenkins/lbg-car-build-app-starter'
        }
    }
    stage('Compile back end') {
        steps {
            //Run Maven on Unix agent
            sh 'cd ./lbg-car-spring-app-starter'
            sh "mvn clean compile"
        }
    }
    stage('Test back end') {        
        steps { 
            sh '''sed '1 s/[^=]*$/dev/' src/main/resources/application.properties'''
            sh 'mvn -D.maven.compile.skip test'
            sh '''sed '1 s/[^=]*$/prod/' src/main/resources/application.properties'''}         
    }
    // maybe delete
    // stage('Package back end') {           
    //    steps { sh 'mvn -D.maven.test.skip -D.maven.compile.skip package'}       
    //}
    stage('Install front end') {
        steps {
            // Install the ReactJS dependencies
            sh 'cd ./lbg-car-react-app-starter'
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
                withCredentials([usernamePassword(credentialsId: env.registryCredentials, passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {

                     sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                     sh 'docker-compose push'
                     }
            }
        }
    }
    
    stage ("Deploy to Server"){
        steps{
            sh 'scp -i ./id_rsa ./docker-compose.yaml jenkins@35.206.178.221:/home/jenkins'
            sh '''ssh -i ./id_rsa jenkins@35.206.178.221 <<EOF
            docker-compose up -d
            '''
        }
    }

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