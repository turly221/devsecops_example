pipeline {
    agent {
        docker {
            image 'python:3.11-slim-buster'
        }
    }
    environment {
        LOG_LEVEL = 'debug'
        // add in jenkins credentials
        SCANTIST_IMPORT_URL = "https://api-v4staging.scantist.io/v2/scans/ci-scan/"
        SCANTISTTOKEN = credentials('SCANTISTTOKEN')
    }
    stages {
        stage('linting and unit testing') {
            steps {
                sh 'pip install -r requirements/dev.txt'
                sh 'flask lint -c'
                sh 'flask test'
            }
        }
        stage('sca scan') {
            steps {
                sh '''
                export DEVSECOPS_IMPORT_URL=$SCANTIST_IMPORT_URL
                export DEVSECOPS_TOKEN=$SCANTISTTOKEN
                curl https://download.scantist.io/sca-bom-detect-v4.5.jar -o scantist-bom-detect.jar
                java -jar scantist-bom-detect.jar -pipRequirementFile prod.txt
                '''
            }
        }
        stage('publish to docker registry') {
            steps {
                sh 'docker push'
            }
        }
        stage('deployment') {
            steps {
                sh 'ssh and deploy'
            }
        }
    }
}
