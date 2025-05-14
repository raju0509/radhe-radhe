pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
    STACK_NAME = 'MyVPCStack-1'
    CHANGE_SET_NAME = 'SafeChangeSet'
  }

  stages {
    stage('Checkout') {
      steps {
        // Replace with correct branch name if not 'master'
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

    stage('Create Change Set') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
            aws cloudformation create-change-set \
              --stack-name ${STACK_NAME} \
              --template-body file://jaimax-devops.yaml \
              --change-set-name ${CHANGE_SET_NAME} \
              --capabilities CAPABILITY_NAMED_IAM \
              --region ${AWS_REGION}
              # --parameters ParameterKey=KeyName,ParameterValue=MyKey
          '''
        }
      }
    }

    stage('Wait for Change Set Creation') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
          sh '''
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
