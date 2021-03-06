pipeline {
  agent any
  tools {
    maven 'maven'
  }
  
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('git-checkout') {
      steps {
        git 'https://github.com/adarshreddy24/webapp.git'
      }
    }

    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json  https://github.com/adarshreddy24/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }

    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/adarshreddy24/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/workspace/DevSecOps_Pipeline/odc-reports/dependency-check-report.xml'
        
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
    }
  }
    
    stage ('Deploy-To-Tomcat') {
            steps {
             sshagent(['tomcat']) {
                  sh 'cp target/*.war /opt/apache-tomcat-9.0.44/webapps/webapp.war'
           }       
         }
    }
    
    stage ('DAST') {
        steps {
         sh "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://10.0.0.46:8081/webapp/"
        }
    }
    
  }
}
