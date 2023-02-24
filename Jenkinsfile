#!/usr/bin/env groovy
 // Liquibase declarative pipeline

pipeline {
  agent {
     docker { image 'liquibase/liquibase:latest' }
   }
  options {
    disableConcurrentBuilds()
  }
  parameters {
    choice(name: 'action', choices: ['dry-run','update','update-to-tag','rollback'], description: '')
    choice(name: 'env', choices: ['DEV', 'LIVE'], description: 'Choose deploy environment')
    string(name: 'db_version', defaultValue: '', description: 'DB tag version')
  }  
 
  environment {
    PROJ="liquibase"
    ENVIRONMENT_STEP="${params.step}"
    BRANCH="${params.pipeline}"
  }

  stages {
    stage('Initialize'){
      steps{
        echo "Choose deploy Action: ${params.action}"
        echo "Set DB tag version: ${params.db_version}"
        echo "Choose deploy environment: ${params.env}"
        script {
        loadEnvironmentVariablesFromFile(".env")
        if (env.env == 'LIVE'){
            env.DB_ENV = env.POSTGRES_PORT_LIVE
        } else {
            env.DB_ENV = env.POSTGRES_PORT_DEV
            }
        } 
      }
    }

    stage('Validation') {
      steps {
        sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} validate'
      }  
    }

    stage('Deploy') { 
      steps {
        script {
          if (env.action == 'update') {
            echo "------liquibase update------"
            sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} tag ${BUILD_NUMBER}'
            sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} update'
          }
          else if (env.action == 'update-to-tag') {
            echo "------liquibase update-to-tag------"
            sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} tag ${BUILD_NUMBER}'
            sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} update-to-tag "${DB_VERSION}"'
          }
          else if (env.action == 'rollback') {
            echo "------liquibase rollback-to-tag------"
            sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} rollback --tag="${DB_VERSION}"' 
          }
          else {
            echo "------liquibase dry-run------"
            sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} status --log-level=INFO'
            sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} update-sql'

          }
        }
      }
    }

    stage('Report') {
      steps {
        echo "------liquibase generate report------"
        sh 'liquibase --defaults-file=liquibase.properties --url="jdbc:postgresql://host.docker.internal:${DB_ENV}/${POSTGRES_DB}" --username=${POSTGRES_USER} --password=${POSTGRES_PASSWORD}  --driver=${LIQUIBASE_DRIVER} db-doc changelogDocs'
      } 
    }  
  } 

  post {
    unstable {
      cleanWs()
    }
    failure {
      cleanWs()
    }
  } 
}

private void loadEnvironmentVariablesFromFile(String path) {
  def file = readFile(path)
  file.split('\n').each { envLine ->
      def (key, value) = envLine.tokenize('=')
      env."${key}" = "${value}"
  }
}