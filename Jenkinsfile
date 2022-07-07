pipeline{
    agent any
    environment {
        report = '$WORKSPACE/Email/email-template.html'
    }
    tools {
        maven "mvn"
        
    }
    stages
    {
       stage('GetCode'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/RL1.0']], extensions: [], userRemoteConfigs: [[credentialsId: 'Git_Yileu', url: 'https://github.com/Yileu/cicd.git']]])
            }
         }        
       stage('Build'){
            steps{
                sh 'mvn clean install'
    
            }
         }
        stage('SonarQube analysis') {
        steps{
        withSonarQubeEnv('sonarqube-9.4') { 
        sh "mvn sonar:sonar"
    }
        }
        }
        stage("Quality Gate") {
            steps{
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
            }
      }      
       stage('Deploy'){
            steps{
                sh '''cd $WORKSPACE
                sudo cp -rf $WORKSPACE/src/main/resources/* /var/www/html/triet.com/ 
                docker restart httpd 
                sh /home/triet/start_email.sh
                '''
            }
         }
    }
    post {
       always {
            script {
                html_body = sh(script: "cat ${report}", returnStdout: true).trim()
                emailext body: "$html_body", 
                subject: '$PROJECT_NAME - Build#$BUILD_NUMBER - $BUILD_STATUS!', 
                to: 'trietpl1999@gmail.com',
                mimeType: 'text/html'
            }
       }
    }
}
