pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
    STACK_NAME = 'MyVPCStack'
    CHANGE_SET_NAME = 'SafeChangeSet'
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

    stage('Ensure Stack Exists') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
            set -e
            echo "Checking if stack '${STACK_NAME}' exists..."
            if ! aws cloudformation describe-stacks --stack-name ${STACK_NAME} --region ${AWS_REGION} > /dev/null 2>&1; then
              echo "Stack does not exist. Creating stack '${STACK_NAME}'..."
              aws cloudformation create-stack \
                --stack-name ${STACK_NAME} \
                --template-body file://jaimax-devops.yaml \
                --capabilities CAPABILITY_NAMED_IAM \
                --region ${AWS_REGION}
              echo "Waiting for stack creation to complete..."
              aws cloudformation wait stack-create-complete \
                --stack-name ${STACK_NAME} \
                --region ${AWS_REGION}
            else
              echo "Stack '${STACK_NAME}' already exists."
            fi
          '''
        }
      }
    }

    stage('Create Change Set') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
            echo "Creating change set..."
            aws cloudformation create-change-set \
              --stack-name ${STACK_NAME} \
              --template-body file://jaimax-devops.yaml \
              --change-set-name ${CHANGE_SET_NAME} \
              --capabilities CAPABILITY_NAMED_IAM \
              --region ${AWS_REGION}
          '''
        }
      }
    }

    stage('Wait for Change Set Creation') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
            echo "Waiting for change set creation..."
            aws cloudformation wait change-set-create-complete \
              --change-set-name ${CHANGE_SET_NAME} \
              --stack-name ${STACK_NAME} \
              --region ${AWS_REGION}
          '''
        }
      }
    }

    stage('Execute Change Set') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
            echo "Executing change set..."
            aws cloudformation execute-change-set \
              --change-set-name ${CHANGE_SET_NAME} \
              --stack-name ${STACK_NAME} \
              --region ${AWS_REGION}
          '''
        }
      }
    }

    stage('Wait for Stack Update') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
            echo "Waiting for stack update to complete..."
            aws cloudformation wait stack-update-complete \
              --stack-name ${STACK_NAME} \
              --region ${AWS_REGION}
          '''
        }
      }
    }

    stage('Delete Change Set') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
            echo "Deleting change set..."
            aws cloudformation delete-change-set \
              --change-set-name ${CHANGE_SET_NAME} \
              --stack-name ${STACK_NAME} \
              --region ${AWS_REGION}
          '''
        }
      }
    }
  }
}
