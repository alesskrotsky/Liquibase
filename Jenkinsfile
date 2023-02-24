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
    choice(name: 'action', choices: ['dry-run','update','update-to-tag','rollback-to-tag'], description: 'Choose DB action type')
    choice(name: 'env', choices: ['DEV', 'LIVE'], description: 'Choose deploy environment')
    string(name: 'db_version', defaultValue: '', description: 'DB tag version. Have to be specified for (update-to-tag) or (rollback-to-tag) actions.')
  }  
 
  environment {
    PROJ_NAME="Liquibase"
    ENVIRONMENT_STEP="${params.step}"
    BRANCH="${params.pipeline}"
  }

  stages {
    stage('Initialize'){
      steps{
        script {
        loadEnvironmentVariablesFromFile(".env")
        if (env.env == 'LIVE'){
            env.DB_ENV = env.POSTGRES_PORT_LIVE
            env.DB_CONTEXT = "live"
        } else {
            env.DB_ENV = env.POSTGRES_PORT_DEV
            env.DB_CONTEXT = "dev"
            }
        env.DB_VERSION = env.db_version
        
        env.DB_URL = "jdbc:postgresql://host.docker.internal:$DB_ENV/$POSTGRES_DB"
        }
        echo "Choose deploy Action: ${params.action}"
        echo "Set DB tag version: ${DB_VERSION}"
        echo "Choose deploy environment: ${DB_ENV}"
      }
    }

    stage('ChangeLog Validation') {
      steps {
        sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER validate"
      }  
    }

    stage('ChangeLog Deploy') {
      steps {
        script {
          if (env.action == 'update') {
            echo "------liquibase update------"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER update-sql --contexts=$DB_CONTEXT"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER update --contexts=$DB_CONTEXT"
          }
          else if (env.action == 'update-to-tag') {
            echo "------liquibase update-to-tag------"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER update-to-tag-sql $DB_VERSION --contexts=$DB_CONTEXT"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER update-to-tag $DB_VERSION --contexts=$DB_CONTEXT"
          }
          else if (env.action == 'rollback-to-tag') {
            echo "------liquibase rollback-to-tag------"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER rollback-sql --tag=$DB_VERSION --contexts=$DB_CONTEXT"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER rollback --tag=$DB_VERSION --contexts=$DB_CONTEXT"
          }
          else {
            echo "------liquibase dry-run------"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER status --contexts=$DB_CONTEXT --log-level=INFO"
            sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER update-sql --contexts=$DB_CONTEXT"
          }
        }
      }
    }

    stage('Report Generation') {
      steps {
        echo "------liquibase generate report------"
        sh "liquibase --defaults-file=liquibase.properties --url=$DB_URL --username=$POSTGRES_USER --password=$POSTGRES_PASSWORD --driver=$LIQUIBASE_DRIVER db-doc changelogDocs"
      } 
    }  
  } 

  post {
    always {
      publishHTML target: [
        allowMissing: false,
        alwaysLinkToLastBuild: false,
        keepAll: true,
        reportDir: 'changelogDocs',
        reportFiles: 'index.html',
        reportName: 'LiquibaseDeploymentReport'
      ]
    }
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