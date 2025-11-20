pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'Maven3'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/smailch/StudentsManagement-DevOps.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
    }
}
