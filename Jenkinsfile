pipeline {
  agent any
  stages {
    stage('Create AMI') {
      steps {
        sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType}'''
        script {
          echo "Entered DeployName: ${DeployName}, AMI_ID: ${AMIId}, Instance Type: ${InstanceType}"
        }

      }
    }
    stage('Test AMI') {
      steps {
        echo 'Tests go here'
      }
    }
    stage('Log Results') {
      steps {
        echo 'Verifying s3 log file'
        sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py verifyLogFile ${DeployName} ${AMIId} ${InstanceType}'''
      }
    }
    stage('Deploy') {
      steps {
        catchError()
      }
    }
  }
  parameters {
    string(name: 'AMIId', defaultValue: 'None', description: 'Enter an AMI ID to boot up and test')
    string(name: 'InstanceType', defaultValue: 't2.micro', description: 'Enter an instance type ex: t2.micro, t2.small, etc..')
    string(name: 'DeployName', defaultValue: 'None', description: 'Enter a unique name for your instances, used for unique logging files')
  }
}