pipeline {
  agent any
  stages {
    stage('Create AMI') {
      parallel {
        stage('Create AMI') {
          steps {
            sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} createAMI'''
            script {
              echo "Entered DeployName: ${DeployName}, AMI_ID: ${AMIId}, Instance Type: ${InstanceType}"
            }

          }
        }
      }
    }
    stage('Wait for AMI to boot') {
      steps {
        sh '''cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} testInstance isRunning'''
      }
    }
    stage('Test AMI') {
      parallel {
        stage('Test AMI') {
          steps {
            sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} testInstance test'''
          }
        }
        stage('test2') {
          steps {
            sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} testInstance a'''
          }
        }
        stage('test') {
          steps {
            sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} testInstance b'''
          }
        }
      }
    }
    stage('Log Results') {
      steps {
        echo 'Verifying s3 log file'
        sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} verifyLogFile'''
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