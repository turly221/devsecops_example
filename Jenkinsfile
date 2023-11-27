pipeline {
    agent {
        docker {
            image 'python:3.11-slim-buster'
            args '-u root --privileged'
        }
    }
    environment {
        LOG_LEVEL = 'debug'
        // add in jenkins credentials
        SCANTIST_IMPORT_URL = "https://api-v4staging.scantist.io/v2/scans/ci-scan/"
        // SCANTISTTOKEN = credentials('SCANTISTTOKEN')
        FLASK_APP = "autoapp.py"
        DATABASE_URL = "sqlite:////tmp/dev.db"
        SECRET_KEY = "not-so-secret"
        SEND_FILE_MAX_AGE_DEFAULT = "0"

    }
    stages {
        stage('linting and unit testing') {
            steps {
                sh '''
                python -m venv .venv
                . .venv/bin/activate
                pip install -r requirements/dev.txt
                apt-get update && apt-get install -y wget
                flask lint -c
                flask test
                '''
            }
        }
        stage('sca scan') {
            steps {
                sh '''
                export DEVSECOPS_IMPORT_URL=$SCANTIST_IMPORT_URL
                export DEVSECOPS_TOKEN=$SCANTISTTOKEN
                wget -O sca-bom-detect-v4.5.jar https://download.scantist.io/sca-bom-detect-v4.5.jar
                java -jar sca-bom-detect-v4.5.jar -pipRequirementFile prod.txt
                '''
            }
        }
        // stage('publish to docker registry') {
            
        //     agent {
        //         label 'build-agent'
        //     }
        //     when {
        //         tag 'v*'
        //     }
        //     steps {
        //         // https://github.com/scantist/jenkins-shared-lib/blob/main/vars/dockerBuildPush.groovy
        //         dockerBuildPush('HWC', 'scantist-images/backend', 'Dockerfile', '--no-cache --target production', '.', 'stable')
        //     }
        // }
        stage('deployment') {
            // agent {
            //     label 'deploy-agent'
            // }
            when {
                tag 'v*'
            }
            steps {
                sh '''
                    sed "s|latest|$TAG_NAME|g" ci/saas/docker-compose.saas.yml > docker-compose.saas.yml
                    rsync docker-compose.saas.yml prod:/home/ubuntu/backend/docker-compose.saas.yml
                    ssh prod <<'ENDSSH'
                    cd /home/ubuntu/backend
                    docker-compose -f docker-compose.saas.yml down --rmi all
                    docker-compose -f docker-compose.saas.yml up -d
                '''
            }
        }
    }
}
