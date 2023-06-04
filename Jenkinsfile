pipeline {
  agent any

  stages {
    stage("Initial cleanup") {
      steps {
        dir("${WORKSPACE}") {
          deleteDir()
        }
      }
    }

    stage('Checkout SCM') {
      steps {
        git branch: 'main', url: 'https://github.com/Vectorjay/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
        sh 'mv .env.sample .env'
        sh 'composer install'
        sh 'php artisan migrate'
        sh 'php artisan db:seed'
        sh 'php artisan key:generate'
      }
    }

    stage('Execute Unit Test') {
      steps {
        sh './vendor/bin/phpunit'
      }
    }

    stage('Code Analysis') {
      steps {
        sh 'phploc app/ --log-csv build/logs/phploc.csv'
      }
    }

    stage('Plot Code Coverage Report') {
      steps {
        script {
          plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [
            [
              displayTableFlag: false,
              exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)',
              file: 'build/logs/phploc.csv',
              inclusionFlag: 'INCLUDE_BY_STRING',
              url: ''
            ]
          ], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'

          // Add the remaining plot steps here
          // ...

        }
      }
    }
    stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }
    
    stage('Upload Artifact to Artifactory') {
      steps {
        script {
          def server = Artifactory.server 'Artifactory-Server'
          def uploadSpec = """{
            "files": [
              {
                "pattern": "php-todo.zip",
                "target": "vector/php-todo",
                "props": "type=zip;status=ready"
              }
            ]
          }"""

          server.upload spec: uploadSpec
        }
      }
    }
    stage ('Deploy to Dev Environments') {
    steps {
    build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }
  }
}

