pipeline {
  agent any
  stages {
    stage('Create AMI') {
      steps {
        sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkinsPythonScript
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
        echo 'test'
        s3Upload(file: 'test.txt', bucket: 'jenkins-log-scratch', path: '/', acl: 'BucketOwnerFullControl')
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