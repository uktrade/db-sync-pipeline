pipeline {

  agent {
    node {
      label env.CI_SLAVE
    }
  }

  options {
    timestamps()
  }

  parameters {
    string(defaultValue: '', description:'Please choose your source: ', name: 'SRC')
    string(defaultValue: '', description:'Please choose your destination: ', name: 'DEST')
  }

  stages {

    stage('Init') {
      steps {
        script {
          validateDeclarativePipeline("${env.WORKSPACE}/Jenkinsfile")
          deployer = docker.image("ukti/deployer:${env.GIT_BRANCH.split("/")[1]}")
          deployer.pull()
        }
      }
    }

    stage('Sync') {
      steps {
        script {
          ansiColor('xterm') {
            deployer.inside {
              tempfile = sh(script: "date +%s", returnStdout: true).trim()
              src = env.SRC.split("/")
              dest = env.DEST.split("/")
              withCredentials([usernamePassword(credentialsId: env.GDS_PAAS_CREDENTIAL, passwordVariable: 'gds_pass', usernameVariable: 'gds_user')]) {
                sh "cf login -a ${env.GDS_PAAS} -u ${gds_user} -p ${gds_pass} -o ${src[0]} -s ${src[1]}"
              }
              sh "cf conduit ${src[2]} -- pg_dump -O -x -f ${src[2]}-${tempfile}.sql"
              sh "cf target -o ${dest[0]} -s ${dest[1]}"
              sh "cf conduit ${dest[2]} -- psql < ${src[2]}-${tempfile}.sql"
            }
          }
        }
      }
    }

  }

  post {
    always {
      script{
        deleteDir()
      }
    }
  }

}
