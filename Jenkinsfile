def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]


pipeline {
    agent any
    
    
    environment{
        
        WORKSPACE = "${env.WORKSPACE}"
        
    }
    
    tools{
        maven 'localMaven'
        jdk 'localJdk'
    }

    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/PromiseMb/Jenkins-Ci-CD-Pipeline-Project.git'
                
            }
        }
        
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn clean package'
                
            }
        
            post {
                success {
                    echo 'archiving...'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
        
        
    stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
    }
    stage('Integration Test'){
        steps {
          sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('Checkstyle Code Analysis'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    
    
    stage ('SonarQube scanning'){
        steps {
            
            withSonarQubeEnv('SonarQube') {
            
            sh """
            mvn sonar:sonar \
              -Dsonar.projectKey=JavaWebApp \
              -Dsonar.host.url=http://172.31.15.131:9000 \
              -Dsonar.login=77b3d75012db6eff7c45cc2cdeb025d6b5cc3b3f
            
            """
            }
        }
    }
    
    stage("Quality Gate"){
        
      steps{
          
       waitForQualityGate abortPipeline: true
       
         }
    }
    
    
    stage("Upload artifact to Nexus"){
        
      steps{
          
       sh 'mvn clean deploy -DskipTests'
       
         }
    }
    
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh "sudo ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
     
     
     stage('Deploy to STAGE env') {
      environment {
        HOSTS = "stage"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
     
     stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    }
     
     
     stage('Deploy to PROD env') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
    
    }
    
    post {
        always {
            echo 'I will always say Hello again!'
            slackSend channel: '#glorious-w-f-devops-alerts', color: 'COLOR_MAP[currentBuild.currentResult]', message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL} \n hi from JJ"
        }
    }
}

