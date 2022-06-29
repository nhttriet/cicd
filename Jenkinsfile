pipeline{
    agent any
    environment {
        PATH = "$PATH:/opt/maven/bin"
        report = '$WORKSPACE/Email/email-template.html'
    }
    tools {
        maven "mvn"
        
    }
    stages
    {
       stage('GetCode'){
            steps{
                git 'https://github.com/Yileu/cicd.git'
                sh 'mvn --version'
                sh 'java -version'
            }
         }        
       stage('Build'){
            steps{
                sh 'mvn clean package'
                // sh 'mvn clean install'
    
            }
         }
        stage('SonarQube analysis') {
        steps{
        withSonarQubeEnv('sonarqube-9.4') { 
        sh "mvn sonar:sonar"
    }
        }
        }
//         stage("Quality Gate") {
//             steps{
//         timeout(time: 1, unit: 'HOURS') {
//             waitForQualityGate abortPipeline: true
//         }
                
//             }
//   }      
       stage('Deploy'){
            steps{
                sh '''cd $WORKSPACE
                pwd
                sudo cp -rf $WORKSPACE/src/main/resources/static/* /var/www/html/triet.com/ 
                #sudo cp -rf $WORKSPACE/src/main/resources/static/index_files /var/www/html/triet.com/
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
