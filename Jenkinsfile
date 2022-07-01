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
                git branch: 'RL1.0', credentialsId: 'Git_Yileu', url: 'https://github.com/Yileu/cicd.git'
                sh 'mvn --version'
                sh 'java -version'
            }
         }        
       stage('Build'){
            steps{
                sh 'mvn clean package'
                sh 'mvn clean install'
    
            }
         }
        stage('SonarQube analysis') {
        steps{
        withSonarQubeEnv('sonarqube-9.4') { 
//         sh "mvn sonar:sonar"
        sh "mvn properties:read-project-properties sonar:sonar"
    }
        }
        }
        stage("Quality Gate") {
            steps{
        timeout(time: 1, unit: 'HOURS') {
            waitForQualityGate abortPipeline: true
        }
                error "Test result is ${qualityGate.status}" 
                error "Test result1 is ${waitForQualityGate}"
                error "Test result2 is ${abortPipeline}"
            }
  }      
//         stage("Quality Gate"){
//             steps{
//                  timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
//             }
//                 def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
//             if (qg.status != 'OK') {
//                 error "Pipeline aborted due to quality gate failure: ${qg.status}"
//             }
//           }
//         }
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
