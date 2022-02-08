pipeline{
    agent any
    parameters{
        string(name: 'CLIENTID', defaultValue: '', description: 'enter the clientid')
        string(name: 'PASSWORD', defaultValue: '', description: 'enter the password')
        string(name: 'DBUSER', defaultValue: '', description: 'enter the secreat value')
        string(name: 'DBPSW', defaultValue: '', description: 'enter the secreat value')
    }
    environment{
        IMAGE_VERSION = "${env.BUILD_NUMBER}"
        IMAGE_NAME = "gcr.io/env-cloudrun/flask-main:v1.${IMAGE_VERSION}"
        SA = "env-cloudrun-sa@env-cloudrun.iam.gserviceaccount.com"
        KEY_FILE = "env-cloudrun-cerdential.json"
    }
    stages{
        stage('git checkout'){
            steps{
                git credentialsId: 'git_id', url: 'https://github.com/ArunAkil/flask-demo.git'
            }
        }
        stage('build image'){
            steps{
                sh "docker build --tag ${IMAGE_NAME} . --file=Dockerfile"
            }
        }
        stage('push container registry'){
            steps{
                sh "gcloud auth activate-service-account ${SA} --key-file=${KEY_FILE}"
                sh "gcloud auth configure-docker"
                sh "docker push ${IMAGE_NAME}"
            }
        }
        stage('deploy'){
            steps{
                sh "gcloud config set project env-cloudrun"
                sh "gcloud auth activate-service-account ${SA} --key-file=${KEY_FILE}"
                sh "gcloud run deploy flask-demo-${IMAGE_VERSION} --image ${IMAGE_NAME} --service-account=${SA} --set-env=clientid=${CLIENTID},password=${PASSWORD} --set-secrets=dbusername=${DBUSER}:${IMAGE_VERSION},password=${DBPSW}:${IMAGE_VERSION} --platform=managed --region=us-central1 --port=5000 --revision-suffix=${IMAGE_VERSION}"
            }
        }
    }
}