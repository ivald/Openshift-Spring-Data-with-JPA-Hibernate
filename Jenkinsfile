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
        stage('Build Image') {
            steps {
                sh "oc start-build openshift --from-file=target/openshift-0.0.1-SNAPSHOT.jar --follow"
            }
        }
        stage('Deploy Image') {
            steps {
                openshiftDeploy depCfg: 'openshift'
            }
        }
    }
}
