pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }
    
    environment {
        HELM_HOME = "/var/jenkins_home/bin"
        PATH = "${HELM_HOME}:${env.PATH}"

        KUBECTL_HOME = tool name: 'kubectl'
        PATH = "${KUBECTL_HOME}/bin:${env.PATH}"
    }
    stages {
        stage('Check Helm') {
            steps {
                sh 'helm version'
            }
        }
        stage('Check kubectl') {
            steps {
                sh 'kubectl version --client'
                sh 'kubectl config current-context' // What is our kubectl context?
            }
        }
        stage('Source'){
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', url: 'https://github.com/SamuelSnowball/spring-boot-hello-world.git'
            }    
        }
        stage('Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "pwd" // Where are we?
                sh "mvn clean verify"
                sh 'docker build -t hello-world-spring .'

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'target/*.jar'
                }
            }
        }
    }
}
