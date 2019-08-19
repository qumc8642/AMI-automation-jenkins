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
        SSH_CREDS = credentials('jenkins-scratch')
        PRIVATE_IP = sh(script: """
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      cd ~/../../../
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      cd home/
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      cd jenkins
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} testInstance grabIP
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      """, returnStdout: true)
      }
      parallel {
        stage('Grab Shelling IP') {
          steps {
            script {
              if (env.PRIVATE_IP == "Instance not found")
              {
                error("The Instance booted cannot be found, IP grab failed")
              } else {
                echo "Private IP of AMI Instance: ${PRIVATE_IP} now shelling for tests"
              }
            }

          }
        }
      }
    }
    stage('Test remote instance') {
      steps {
        sh '''cd ~/../../../
cd home/
cd jenkins
aws ssm send-command         --targets "Key=Name,Values=${DeployName}"         --document-name "AWS-RunShellScript"         --parameters commands=["bash /home/jenkins/testscript.sh"]         --comment "Run unit test sh script"     --output-s3-bucket-name "jenkins-log-scratch"'''
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
        echo 'AMI successfully deployed on AWS Scratch Environment'
      }
    }
  }
  post {
    failure {
      echo 'Verifying s3 log file'
      sh '''echo Navigating to correct directory
cd ~/../../../
cd home/
cd jenkins
python3 AMICreatePython.py ${DeployName} ${AMIId} ${InstanceType} verifyLogFile'''
      googlechatnotification(url: 'https://chat.googleapis.com/v1/spaces/AAAArsfukF8/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=QhVdpCSzJ9_2jmJL6R_NLAqcA5UOCf0WUNlJ2Farb7c%3D', message: 'AMI creation and testing failed.  Find the log file here: https://s3.console.aws.amazon.com/s3/buckets/jenkins-log-scratch/?region=us-east-1&tab=overview', notifyAborted: 'false', notifyFailure: 'false', notifyNotBuilt: 'false', notifySuccess: 'false', notifyUnstable: 'false', notifyBackToNormal: 'false', suppressInfoLoggers: 'false', sameThreadNotification: 'false')

    }

    success {
      googlechatnotification(url: 'https://chat.googleapis.com/v1/spaces/AAAArsfukF8/messages?key=AIzaSyDdI0hCZtE6vySjMm-WEfRq3CPzqKqqsHI&token=QhVdpCSzJ9_2jmJL6R_NLAqcA5UOCf0WUNlJ2Farb7c%3D', message: 'AMI creation and testing succeeded.  Find the log file here: https://s3.console.aws.amazon.com/s3/buckets/jenkins-log-scratch/?region=us-east-1&tab=overview', notifyAborted: 'false', notifyFailure: 'false', notifyNotBuilt: 'false', notifySuccess: 'false', notifyUnstable: 'false', notifyBackToNormal: 'false', suppressInfoLoggers: 'false', sameThreadNotification: 'false')

    }

  }
  parameters {
    string(name: 'AMIId', defaultValue: 'None', description: 'Enter an AMI ID to boot up and test')
    string(name: 'InstanceType', defaultValue: 't2.micro', description: 'Enter an instance type ex: t2.micro, t2.small, etc..')
    string(name: 'DeployName', defaultValue: 'None', description: 'Enter a unique name for your instances, used for unique logging files')
  }
}