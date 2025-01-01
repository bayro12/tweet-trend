def registry = 'https://bayraktarkaragoz.jfrog.io'

pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
    }

    stages {
        stage("Build") {
            steps {
                echo "----------- Build Started ----------"
                sh 'mvn clean package -Dmaven.test.skip=true'
                echo "----------- Build Completed ----------"
            }
        }

        stage("Jar Staging") {
            steps {
                echo "----------- Staging JAR Files ----------"
                sh '''
                mkdir -p jarstaging
                cp target/*.jar jarstaging/
                '''
                echo "----------- JAR Files Staged ----------"
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer(url: registry + "/artifactory", credentialsId: "artfiact-cred")
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/*.jar",
                                "target": "bayraktarmaven-libs-release/",
                                "flat": true
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline Completed. Cleaning up workspace."
            cleanWs()
        }
        failure {
            echo "Pipeline Failed. Please check the console output for errors."
        }
    }
}
