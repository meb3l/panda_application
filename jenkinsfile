pipeline {
    agent {
      label 'Slave'
    }
    
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }
    
    stages {
        stage('Checkout GIT') {
            steps {
                git branch: 'feature/selenium', url: 'https://github.com/meb3l/panda_application.git'
            }
        }

        stage('Clear running apps') {
            steps {
                sh 'docker rm -f pandaapp || true'
            }
        }
        stage('Build and Junit') {
            steps {
                sh "mvn clean install"
            }
        }
        stage('Build Docker image'){
            steps {
                sh "mvn package -Pdocker"
            }
        }
        stage('Run Docker app') {
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
                configFileProvider([configFile(fileId: '9d1ed313-ea70-4fa9-9934-7108c53eca75', variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                    sh "mvn -gs $MAVEN_GLOBAL_SETTINGS deploy -Dmaven.test.skip=true -e"
                }
            } 
            post {
                always { 
                    sh 'docker stop pandaapp'
                    deleteDir()
                }
            }
        }
    }
}