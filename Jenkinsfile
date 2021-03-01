pipeline {
  agent none
  stages {
    stage('build maven') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2:z -u root'
          reuseNode true
        }

      }
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'compiling worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('test maven') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2:z -u root'
          reuseNode true
        }

      }
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'running unit tests'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('package maven') {
      agent {
        docker {
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2:z -u root'
          reuseNode true
        }

      }
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'packaging worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('docker package maven') {
      agent any
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'docker package'
        dir(path: 'worker') {
          script {
            docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
              def workerImage = docker.build("lexarflash8g/worker:v${env.BUILD_ID}", "./")
              workerImage.push()
              workerImage.push("${env.BRANCH_NAME}")
            }
          }

        }

      }
    }

    stage('buildnode') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        branch 'master'
      }
      steps {
        echo 'compiling worker app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('testnode') {
      agent {
        docker {
          image 'node:8.16.0-alpine'
        }

      }
      when {
        branch 'master'
      }
      steps {
        echo 'running unit tests'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('docker package node') {
      agent any
      when {
        branch 'master'
      }
      steps {
        echo 'docker package'
        dir(path: 'result') {
          script {
            docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
              def workerImage = docker.build("lexarflash8g/result:v${env.BUILD_ID}", "./")
              workerImage.push()
              workerImage.push("${env.BRANCH_NAME}")
            }
          }

        }

      }
    }

    stage('buildpython') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '-u root'
        }

      }
      steps {
        echo 'compiling worker app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('testpython') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '-u root'
        }

      }
      steps {
        echo 'running unit tests'
        dir(path: 'vote') {
          sh 'pip install nose'
          sh 'nosetests -v'
        }

      }
    }

    stage('docker package python') {
      agent any
      steps {
        echo 'docker package'
        dir(path: 'vote') {
          script {
            docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
              def workerImage = docker.build("lexarflash8g/vote:v${env.BUILD_ID}", "./")
              workerImage.push()
              workerImage.push("${env.BRANCH_NAME}")
            }
          }

        }

      }
    }

    stage('docker-compose up') {
      steps {
        dir(path: 'worker')
      }
    }

  }
  tools {
    nodejs 'node'
  }
}