// This jenkinsfile is used to run CI/CD on my local (Windows) box, no VM's needed.

pipeline {

  agent {
        docker {
            image 'evanmann/ensf400-final-project:projectDependencies'
            args '--network=ci-net -it'
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
        SONAR_TOKEN = credentials('sonarqube-token')  // Getting the stored sonarqube token to connect with
   }

  stages {

    // build the war file (the binary).  This is the only
    // place that happens.

    stage('Checkout') {
      steps {
          script {
              // Change to the 'demo-master' folder before checking out the code
              dir('demo-master') {
                  checkout scm
                  }
              }
          }
    }

    stage('Build') {
        steps {
            script {
                // Change to the 'demo-master' folder before running gradle
                dir('demo-master') {
                    sh 'chmod +x gradlew'
                    sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew clean assemble'
                }
            }
        }
    }

    // run all the unit tests - these do not require anything else
    // to be running and most run very quickly.
    stage('Unit Tests') {
        steps {
            dir('demo-master') {
                sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew test'
            }
        }
        post {
            always {
                junit 'demo-master/build/test-results/test/*.xml'
            }
        }
    }

    // run the tests which require connection to a
    // running database.
    stage('Database Tests') {
      steps {
        dir('demo-master') {
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew integrate'
        }
      }
      post {
        always {
          junit 'demo-master/build/test-results/integrate/*.xml'
        }
      }
    }

    // These are the Behavior Driven Development (BDD) tests
    // See the files in src/bdd_test
    // These tests do not require a running system.
    stage('BDD Tests') {
      steps {
        dir('demo-master') {
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew generateCucumberReports'
          // generate the code coverage report for jacoco
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew jacocoTestReport'
        }
      }
      post {
          always {
            junit 'demo-master/build/test-results/bdd/*.xml'
          }
        }
    }

    // Runs an analysis of the code, looking for any
    // patterns that suggest potential bugs.
    
    stage('Static Analysis') {
      steps {
        dir('demo-master') {
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew sonarqube'
          // wait for sonarqube to finish its analysis
          sleep 5
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew checkQualityGate'
        }
      }
    }
    
    //DOCKER IMAGE NEED SONARQUBE SET UP


    // Move the binary over to the test environment and
    // get it running, in preparation for tests that
    // require a whole system to be running.
    
    stage('Deploy to Test') {
      steps {
        dir('demo-master') {
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew apprun'
          // pipenv needs to be installed and on the path for this to work.
          sh 'PIPENV_IGNORE_VIRTUALENVS=1 pipenv install'

          // Wait here until the server tells us it's up and listening
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew waitForHeartBeat'

          // clear Zap's memory for the incoming tests
          sh 'curl http://zap/JSON/core/action/newSession -s --proxy localhost:9888'
        }
      
      }
    }
    


    // Run the tests which investigate the functioning of the API.
    stage('API Tests') {
      steps {
        dir('demo-master') {
          sh 'export JAVA_HOME=/opt/java/openjdk && ./gradlew runApiTests'
        }
       
      }
      post {
        always {
          junit 'build/test-results/api_tests/*.xml'
        }
      }
    }

    // We use a BDD framework for some UI tests, Behave, because Python rules
    // when it comes to experimentation with UI tests.  You can try things and see how they work out.
    // this set of BDD tests does require a running system.
    // BDD at the UI level is just to ensure that basic capabilities work,
    // not that every little detail of UI functionality is correct.  For
    // that purpose, see the following stage, "UI Tests"
    stage('UI BDD Tests') {
      steps {
        sh './gradlew runBehaveTests'
        sh './gradlew generateCucumberReport'
      }
      post {
        always {
          junit 'build/test-results/bdd_ui/*.xml'
        }
      }
    }

    // This set of tests investigates the functionality of the UI.
    // Note that this is separate fom the UI BDD Tests, which
    // only focuses on essential capability and therefore only
    // covers a small subset of the possibilities of UI behavior.
    stage('UI Tests') {
        steps {
            sh 'cd src/ui_tests/java && ./gradlew clean test'
        }
        post {
            always {
                junit 'src/ui_tests/java/build/test-results/test/*.xml'
            }
        }
    }

    // Run OWASP's "DependencyCheck". https://owasp.org/www-project-dependency-check/
    // You are what you eat - and so it is with software.  This
    // software consists of a number of software by other authors.
    // For example, for this project we use language tools by Apache,
    // password complexity analysis, and several others.  Each one of
    // these might have security bugs - and if they have a security
    // bug, so do we!
    //
    // DependencyCheck looks at the list of known
    // security vulnerabilities from the United States National Institute of
    // Standards and Technology (NIST), and checks if the software
    // we are importing has any major known vulnerabilities. If so,
    // the build will halt at this point.
    stage('Security: Dependency Analysis') {
      steps {
         sh './gradlew dependencyCheckAnalyze'
      }
    }

    // Run Jmeter performance testing https://jmeter.apache.org/
    // This test simulates 50 users concurrently using our software
    // for a set of common tasks.
    stage('Performance Tests') {
      steps {
         sh './gradlew runPerfTests'
      }
    }

    // Runs mutation testing against some subset of our software
    // as a spot test.  Mutation testing is where bugs are seeded
    // into the software and the tests are run, and we see which
    // tests fail and which pass, as a result.
    //
    // what *should* happen is that where code or tests are altered,
    // the test should fail, shouldn't it? However, it sometimes
    // happens that no matter how code is changed, the tests
    // continue to pass, which implies that the test wasn't really
    // providing any value for those lines.
    stage('Mutation Tests') {
      steps {
         sh './gradlew pitest'
      }
    }

    stage('Build Documentation') {
      steps {
         sh './gradlew javadoc'
      }
    }

    stage('Collect Zap Security Report') {
      steps {
        sh 'mkdir -p build/reports/zap'
        sh 'curl http://zap/OTHER/core/other/htmlreport --proxy localhost:9888 > build/reports/zap/zap_report.html'
      }
    }


    // This is the stage where we deploy to production.  If any test
    // fails, we won't get here.  Note that we aren't really doing anything - this
    // is a token step, to indicate whether we would have deployed or not.  Nothing actually
    // happens, since this is a demo project.
    stage('Deploy to Prod') {
      steps {
        // just a token operation while we pretend to deploy
        sh 'sleep 5'
      }
    }

  }

}
