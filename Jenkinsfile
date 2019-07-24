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
      environment {
        publicIP = ''
      }
      parallel {
        stage('Grab Shelling IP') {
          steps {
            script {
              def IP = sh(script: """
              cd ~/../../../
              cd home/
              cd jenkins
              python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} testInstance grabIP
              """, returnStdout: true)
              println IP

              def publicIP = new ParametersAction([new StringParameterValue("publicIP", IP)], ["publicIP"])
              build.addAction(publicIP)
            }

          }
        }
        stage('test2') {
          steps {
            script {
              println ${publicIP}
            }

          }
        }
        stage('test') {
          steps {
            script {
              println ${publicIP}
            }

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