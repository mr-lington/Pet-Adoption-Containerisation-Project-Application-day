pipeline{
    agent any
    tools {
    maven 'maven'
  }
  
    stages{
        stage('Pull Source Code from GitHub') {
            steps {
                git branch: 'main',
                credentialsId: 'git',
                url: 'https://github.com/CloudHight/Pet-Adoption-Containerisation-Project-Application-Day-Team.git'
            }
        }
        
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh "mvn sonar:sonar -Dsonar.java.binaries=target/classes"
                }
            }   
        }
        
        stage('Code Quality Approval') {
            steps {
                waitForQualityGate abortPipeline: true,
                credentialsId: 'sonar'
            }
        }

            
        stage('Build Code') {
            steps {
              sh 'mvn package -Dmaven.test.skip'
            }
        }
        
        stage('Send Artifacts to Ansible Server') {
            steps { 
                sshagent(['jenkins']) {
                    sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/petadoption/target/spring-petclinic-2.4.2.war  ec2-user@13.40.121.91:/opt/docker'
                }
            }
        }
        
        stage('Trigger Stage Playbooks') {
            steps { 
                sshagent(['jenkins-key']) {
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@13.40.121.91 "ansible-playbook -i /etc/ansible/hosts /opt/docker/docker-image.yml && ansible-playbook -i /etc/ansible/hosts /opt/docker/docker-stage.yml"'
                }
            }
        }
        stage ('Deploy To Prod'){
            input{
                message "Do you want to proceed for production deployment?"
           }
            steps { 
                sshagent(['jenkins-key']) {
                    sh 'ssh -o StrictHostKeyChecking=no ec2-user@13.40.121.91 "ansible-playbook -i /etc/ansible/hosts /opt/docker/docker-prod.yml"'
                }
            }
        }
    }
}