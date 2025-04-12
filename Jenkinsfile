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
        setGitHubPullRequestStatus (
            state: 'PENDING',
            context: 'Jenkins',
            message: 'Build in progress',
        )
      }
    }
  }

  post {
    failure {
        setGitHubPullRequestStatus (
            state: 'FAILURE',
            context: 'Jenkins',
            message: 'Build failed',
        )
    }

    success {
        setGitHubPullRequestStatus (
            state: 'SUCCESS',
            context: 'Jenkins',
            message: 'Build succeeded',
        )
    }
  }
}
