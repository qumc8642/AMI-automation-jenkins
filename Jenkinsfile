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
        sh 'aws ssm send-command --document-name "AWS-RunShellScript" --document-version "1" --targets \'[{"Key":"tag:Name","Values":["INSERT PARAMETER HERE"]}]\' --parameters \'{"workingDirectory":[""],"executionTimeout":["3600"],"commands":["#package install checks, if the package exists each line will return \'Failed to set locale, defaulting to C\'","","#check \\"packages\\" section in yml script. Only prints when the packages aren\'t found","echo \\"Testing package installs...\\"","for package in zip unzip chrony mdadm xfsprogs xfsdump gpg screen wget sysstat pciutils cryptsetup bash-completion bind-utils epel-release centos-release-scl amazon-ssm-agent;","do if ( ! ( yum info $package |& grep \\"Installed Packages\\" >> output.txt ) )","then echo \\"Error: $package is not installed\\";","fi;","done;","","#check users, groups, and sudo file","echo \\"\\"","echo \\"Testing users, groups, and permissions...\\"","if ( ! ( awk -F\':\' \'{ print $1}\' /etc/passwd |& grep \'tduser\' >> output.txt) )","then echo \\"Error: tduser not found\\"","fi;","","for group in wheel adm systemd-journal","do if ( ! ( groups tduser |& grep $group >> output.txt ) )","then echo \\"Error: group $group not found\\"","fi;","done;","","#test SELinux disable","echo \\"\\"","echo \\"Testing SELinux disable...\\"","if ( ! ( cat /etc/sysconfig/selinux | grep \\"SELINUX=disabled\\" >> output.txt ) )","then echo \\"Error: SELinux not disabled in /etc/sysconfig/selinux\\"","fi;","","if ( ! ( cat /etc/selinux/config | grep \\"SELINUX=disabled\\" >> output.txt ) )","then echo \\"Error: SELinux not disabled in /etc/selinux/config\\"","fi;","","#root user login disable","echo \\"\\"","echo \\"Testing user root login disable...\\"","if ( ! ( cat /etc/ssh/sshd_config | grep \\"PermitRootLogin no\\" >> output.txt ) )","then echo \\"Error: Root login still enabled\\"","fi;","","#testing centos user removal and the tduser cloud formation configuration","echo \\"\\"","echo \\"Testing centos user removal and tduser config...\\"","if (cat /etc/passwd | grep centos >> output.txt )","then echo \\"Error: Centos user still exists\\"","fi;","","if ( ! ( cat /etc/cloud/cloud.cfg | grep \\"name: tduser\\" >> output.txt ) )","then echo \\"Error: Cloud init does not target tduser\\"","fi;","","#testing EPEL and wpa package config","echo \\"\\"","echo \\"Testing package config for EPEL and wpa...\\"","if ( ! ( yum info wpa_supplicant |& grep \\"Available Packages\\" >> output.txt ) )","then echo \\"Error: wpa_supplicant still installed\\"","fi;","","if ( ! ( yum info bwm-ng pigz |& grep \\"Installed Packages\\" >> output.txt ) )","then echo \\"Error: bwm-ng pigz is not installed\\"","fi;","","#testing python configurations","echo \\"\\"","echo \\"Testing python configuration...\\"","if ( ! ( python --version |& grep \\"2.7\\" >> output.txt ) )","then echo \\"Error: python 2.7 is not installed\\"","fi;","python --version","","if ( ! ( pip --version |& grep \\"10.0\\" >> output.txt ) )","then echo \\"Error: pip 10.0 for python 2.7 is not installed\\"","fi;","pip --version","","if ( ! ( python3 --version |& grep \\"3.6\\" >> output.txt ) )","then echo \\"Error: python 2.7 is not installed\\"","fi;","python3 --version","","if ( ! ( pip3 --version |& grep \\"19\\" >> output.txt ) )","then echo \\"Error: python 2.7 is not installed\\"","fi;","pip3 --version","","","#check pip and pip3 symlinks","echo \\"\\"","echo \\"Testing simlinks for pip, python, and packages...\\"","if ! [[ -L \\"/usr/local/bin/pip\\" ]]","then echo \\"Error: pip symlink does not exist\\"","fi;","","if ! [[ -L \\"/usr/bin/pip3\\" ]]","then echo \\"Error: pip3 symlink does not exist\\"","fi;","","if ! [[ -L \\"/usr/bin/aws\\" ]]","then echo \\"Error: pip3 aws symlink does not exist\\"","fi;","","if ! [[ -L \\"/opt/trimble_data/trip/trip.conf\\" ]]","then echo \\"Error: trip symlink dependency does not exist\\"","fi;","","if ! [[ -L \\"/opt/trimble_data/tupac/tupac.conf\\" ]]","then echo \\"Error: tupac /opt/trimble_data/tupac simlink does not exist\\"","fi;","","if ! [[ -L \\"/usr/bin/tupac\\" ]]","then echo \\"Error: tupac /usr/bin/tupac symlink does not exist\\"","fi;","","","#checking python package installs","echo \\"\\"","echo \\"Testing pip and pip3 installs...\\"","for pippackage in pyyaml frozendict cryptography tdags tdec2tools tdsystemtools trip colorama;","do if ( ! ( pip show $pippackage >> output.txt ) )","then echo \\"Error: package $pippackage is not installed\\"","fi;","done;","","","for pippackage in pyyaml frozendict cryptography awscli tdstupac;","do if ( ! ( pip3 show $pippackage >> output.txt ) )","then echo \\"Error: package $pippackage is not installed\\"","fi;","done;","","","echo \\"\\"","echo \\"Testing s3 TData repository configuration file installs...\\"","for config in pip.conf yum.repos.d/artifactory.repo tdags.conf trip.conf tupac.conf","do if (cat /etc/$config |& grep \\"No such file or directory\\" >> output.txt)","then echo \\"Error: $config has not been installed\\"","fi;","done;","","echo \\"\\"","echo \\"Testing Amazon SSM Agent configuration...\\"","if (! (cat /etc/systemd/system/amazon-ssm-agent.service |& grep \\"Restart=always\\" >> output.txt ) )","then echo \\"Error: Restart=always setting not set for amazon ssm, current setting is Restart=on-failure\\"","fi;","","if (! (cat /etc/systemd/system/amazon-ssm-agent.service |& grep \\"RestartSec=15\\" >> output.txt ) )","then echo \\"Error: Restart=always setting not set for amazon ssm, current setting is Restart=on-failure\\"","fi;","","if (! (systemctl | grep \\"loaded active running\\" | grep amazon-ssm-agent >> output.txt ) )","then echo \\"Error: Amazon ssm agent is not currently running\\"","fi;","","echo \\"\\"","echo \\"Installing Tdata bash and ssh utilities\\"","if ( cat /etc/profile.d/tdata_prompt.sh |& grep \\"No such file or directory\\" >> output.txt )","then echo \\"Error: tdata_prompt.sh has not been installed\\"","fi;","","if ( ! ( trip show td-ssh-key-utils |& grep Version: >> output.txt ) )","then echo \\"Error: td-ssh-key-utils has not been installed\\"","fi;","","if ! [[ -L \\"/var/lib/cloud/scripts/per-instance/reset-host-ssh-keys\\" ]]","then echo \\"Error: tupac /usr/bin/tupac symlink does not exist\\"","fi;","","echo \\"\\"","echo \\"Testing ec2-metadata, croudstrike, chronyd...\\"","if ( cat /usr/bin/ec2-metadata |& grep \\"No such file or directory\\" >> output.txt )","then echo \\"Error: ec2-metadata has not been installed\\"","fi;","","if ( ! ( trip show td-install-crowdstrike |& grep Version: >> output.txt ) )","then echo \\"Error: td-install-crowdstrike has not been installed\\"","fi;","","echo \\"\\"","echo \\"Testing development related package deletion...\\"","for package in gcc cpp glibc-devel glibc-headers;","do if ( yum info $package |& grep \\"Installed Packages\\" >> output.txt )","then echo \\"Error: $package is still installed\\";","fi;","done;","","","","","","","","",""]}\' --timeout-seconds 600 --max-concurrency "50" --max-errors "0" --output-s3-bucket-name "jenkins-log-scratch" --region us-east-1'
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