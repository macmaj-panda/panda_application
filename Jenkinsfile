pipeline {
    agent {
        label 'Slave'
    }
    
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "m3"
    }
    
    environment {
        IMAGE=(readMavenPom().getArtifactId())
        VERSION=(readMavenPom().getVersion())
    }
    
    stages {
        
        stage('Clear running apps'){
            steps {
                sh 'docker rm -f pandaapp || true'
            }
        }
        
        stage('Get Code') {
            steps {
                //git branch: 'dev', url: 'https://github.com/macmaj-panda/panda_application.git'
                checout scm
            }
        }
        
        stage('Build and Junit') {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage('Build Docker image'){
            steps {
                sh "mvn package -Pdocker -Dmaven.test.skip=true"
            }
        }
        
        stage('Run Docker app'){
            steps {
                sh "docker run -d -p 0.0.0.0:8080:8080 --name pandaapp -t ${IMAGE}:${VERSION}"
            }
        }
        
        stage('Test Selenium') {
            steps {
                sh "mvn test -Pselenium"
            }
        }
        
        stage('Deploy jar to artifactory') {
            steps {
                configFileProvider([configFile(fileId: '0aabf9fb-d010-4f2e-9601-91b9c40592b2', variable: 'mavensettings')]) {
                sh "mvn -s $mavensettings deploy -Dmaven.test.skip=true -e"
            }
            }
        }
    }
    post {
        always{
        sh 'docker stop pandaapp'
            deleteDir()
            }
            
        }
}
