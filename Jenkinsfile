pipeline {
  agent any

tools {
    nodejs "Nodejs"
}

 environment {
    DOCKER_REGISTRY = "docker.io"
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    VERSION = "${env.BUILD_ID}"
  }

  stages {

 stage('Install Dependencies') {
      steps {

        sh 'npm ci'
      }
    }

    stage('Build Project') {
      steps {
        // Build the Angular project
        sh 'npm run build'
      }
    }


    stage('Docker Build and Push') {
      steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t lhdenis35/frontend:${VERSION} .'
          sh 'docker push lhdenis35/frontend:${VERSION}'
      }
    }


     stage('Cleanup Workspace') {
      steps {
        deleteDir()
      }
    }

     stage('Update Image Tag in GitOps') {
      steps {
         checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'git-ssh', url: 'git@github.com:lhdenis/food-delivery-app-frontend.git']])
        script {
          // Set the new image tag with the Jenkins build number
       sh '''
          sed -i "s/image:.*/image: lhdenis35\\/frontend:${VERSION}/" aws/angular-manifest.yml
        '''

          sh 'git checkout master'
          sh 'git add .'
          sh 'git commit -m "Update image tag"'
        sshagent(['git-ssh'])
            {
                  sh('git push')
            }
        }
      }
    }
  }

  post {
    always {
      cleanWs()              // nettoie le workspace proprement
      sh 'docker builder prune -af || true'   // libère le cache buildkit (optionnel)
    }
  }
}


