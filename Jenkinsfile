def registry = 'https://valaxy031.jfrog.io'

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
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifiact-cred"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "libs-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
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
