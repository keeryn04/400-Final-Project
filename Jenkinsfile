// This jenkinsfile is used to run CI/CD on my local (Windows) box, no VM's needed.

pipeline {

  agent {
        docker {
            image 'evanmann/ensf400-final-project:projectDependencies'
            args '-it'
            //registryUrl 'https://index.docker.io/v1/'
            //registryCredentialsId 'your-credentials-id'
        }
    }
    
  tools {
    jdk 'jdk11'
  }

   environment {
        // This is set so that the Python API tests will recognize it
        // and go through the Zap proxy waiting at 9888
        HTTP_PROXY = 'http://127.0.0.1:9888'
   }

  stages {
    stage('Notify GitHub (pending)') {
      steps {
        githubNotify context: 'Jenkins CI', status: 'PENDING', description: 'Build started...'
      }
    }
  }

  post {
    failure {
      script {
        setGitHubPullRequestStatus context: 'Jenkins CI', status: 'FAILURE'
      }
    }
    success {
      script {
        setGitHubPullRequestStatus context: 'Jenkins CI', status: 'SUCCESS'
      }
    }
  }
}
