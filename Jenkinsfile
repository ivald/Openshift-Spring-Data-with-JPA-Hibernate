pipeline {
    agent any

    stages {
        stage ('Compile Stage') {
            steps {
                withMaven(maven : 'Apache MAVEN 3.3.3') {
                    sh 'mvn clean compile'
                }
            }
        }
        stage ('Testing Stage') {
            steps {
                withMaven(maven : 'Apache MAVEN 3.3.3') {
                    sh 'mvn test'
                }
            }
        }
        stage ('Deployment Stage') {
            steps {
                withMaven(maven : 'Apache MAVEN 3.3.3') {
                    sh 'mvn package -DskipTests'
                }
            }
        }
        stage('Deploy Image') {
            steps {
                openshiftDeploy depCfg: 'openshift'
            }
        }
    }
}