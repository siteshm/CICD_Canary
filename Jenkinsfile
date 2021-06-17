pipeline {
    agent any
    environment {
        PROJECT_ID = 'arctic-robot-278510'
        CLUSTER_NAME = 'cluster-1'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'gke'
    }
    stages {
        stage('Setup parameters') {
            steps {
                script { properties([parameters([string(defaultValue: '2', description: 'maxSurge: The number of pods that can be created above the desired amount of pods during an update', name: 'MaxSurge'), string(defaultValue: '1', description: 'maxUnavailable: The number of pods that can be unavailable during the update process', name: 'MaxUnavailable'), string(defaultValue: '5', description: 'Number of Replicas', name: 'TotalPod')])])
                       }
            }
        }
        stage("Checkout code") {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'GIT_CREDENTIALS', url: 'https://github.com/siteshm/CICD.git']]])
            }
            //git credentialsId: 'GIT_CREDENTIALS', url: 'https://github.com/siteshm/CICD.git'
        }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("siteshm/hello:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }        
        stage('Deploy to Kubernetes cluster - Rolling Update ') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deploy.yaml"
                sh "sed -i 's/MaxSurge/${MaxSurge}/g' deploy.yaml"
                sh "sed -i 's/MaxUnavailable/${MaxUnavailable}/g' deploy.yaml"
                sh "sed -i 's/TotalPod/${TotalPod}/g' deploy.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deploy.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
    }    
}
