pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }
    
    environment {
        HELM_HOME = "/var/jenkins_home/bin"
        PATH = "${HELM_HOME}:${env.PATH}"
    }
    stages {

        stage('Install kubectl') {
            steps {
                sh '''
                curl -LO https://dl.k8s.io/release/v1.28.0/bin/linux/amd64/kubectl
                chmod +x kubectl
                mv kubectl "/var/jenkins_home/bin/kubectl"
                '''
            }
        }
        stage('Check Helm') {
            steps {
                sh 'helm version'
            }
        }
        stage('Check kubectl') {
            steps {
                sh 'kubectl version --client'

                // Set kubectl context to kind-my-cluster if not already set
                script {
                    sh 'kubectl config get-contexts' // What do we have avaliable?
                    
                    def context = sh(
                        script: 'kubectl config current-context || echo "none"',
                        returnStdout: true
                    ).trim()

                    if (context == 'none' || context == '') {
                        echo "No context set — switching to kind-my-cluster"
                        sh 'kubectl config use-context kind-my-cluster'
                    } else {
                        echo "Current context is: ${context}"
                    }
                }
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
