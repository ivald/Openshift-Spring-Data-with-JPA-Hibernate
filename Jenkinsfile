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
            sh "oc start-build openshiftapp --from-file=target/openshift-0.0.1-SNAPSHOT.jar --follow"
        }
        stage('Deploy Image') {
            openshiftDeploy depCfg: 'openshiftapp'
            openshiftVerifyDeployment depCfg: 'openshiftapp', replicaCount: 1, verifyReplicaCount: true
        }
    }
}