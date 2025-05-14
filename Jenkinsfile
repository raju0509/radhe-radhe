pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
    STACK_NAME = 'MyVPCStack-1'
    CHANGE_SET_NAME = 'SafeChangeSet'
    STACK_EXISTS = ''
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'master', url: 'https://github.com/raju0509/radhe-radhe.git'
      }
    }

    stage('Validate CF Template') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh 'aws cloudformation validate-template --template-body file://jaimax-devops.yaml'
        }
      }
    }

    stage('Create or Update Stack') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          script {
            def stackExists = sh(
              script: "aws cloudformation describe-stacks --stack-name ${STACK_NAME} --region ${AWS_REGION}",
              returnStatus: true
            ) == 0

            if (stackExists) {
              env.STACK_EXISTS = 'true'
              echo "Stack exists. Creating change set..."
              sh """
                aws cloudformation create-change-set \
                  --stack-name ${STACK_NAME} \
                  --template-body file://jaimax-devops.yaml \
                  --change-set-name ${CHANGE_SET_NAME} \
                  --capabilities CAPABILITY_NAMED_IAM \
                  --region ${AWS_REGION}
              """
            } else {
              env.STACK_EXISTS = 'false'
              echo "Stack does not exist. Creating stack..."
              sh """
                aws cloudformation create-stack \
                  --stack-name ${STACK_NAME} \
                  --template-body file://jaimax-devops.yaml \
                  --capabilities CAPABILITY_NAMED_IAM \
                  --region ${AWS_REGION}
              """
            }
          }
        }
      }
    }

    stage('Wait for Completion') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          script {
            if (env.STACK_EXISTS == 'true') {
              echo "Waiting for change set to complete..."
              sh """
                aws cloudformation wait change-set-create-complete \
                  --change-set-name ${CHANGE_SET_NAME} \
                  --stack-name ${STACK_NAME} \
                  --region ${AWS_REGION}

                echo "Executing change set..."
                aws cloudformation execute-change-set \
                  --change-set-name ${CHANGE_SET_NAME} \
                  --stack-name ${STACK_NAME} \
                  --region ${AWS_REGION}

                echo "Waiting for stack update to complete..."
                aws cloudformation wait stack-update-complete \
                  --stack-name ${STACK_NAME} \
                  --region ${AWS_REGION}
              """
            } else {
              echo "Waiting for stack creation to complete..."
              sh """
                aws cloudformation wait stack-create-complete \
                  --stack-name ${STACK_NAME} \
                  --region ${AWS_REGION}
              """
            }
          }
        }
      }
    }

    stage('Delete Change Set (if any)') {
      when {
        expression { env.STACK_EXISTS == 'true' }
      }
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh """
            aws cloudformation delete-change-set \
              --change-set-name ${CHANGE_SET_NAME} \
              --stack-name ${STACK_NAME} \
              --region ${AWS_REGION}
          """
        }
      }
    }
  }
}
