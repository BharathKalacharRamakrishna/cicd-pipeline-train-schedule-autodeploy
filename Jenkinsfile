pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER = credentials ('bharath_DockerHub')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') 
        {
      //      when {
      //          branch 'master'
      //     }
            steps {
                script {
                  sh 'sudo docker image build -t bharath1308/train-schedule:$BUILD_NUMBER .'
                    }
                }
        }
        stage('Push Docker Image') {
      //      when {
      //          branch 'master'
      //      }
            steps {
                script {
                    sh '''
                        sudo docker login --username $DOCKER_USR --password $DOCKER_PSW
                        sudo docker push bharath1308/train-schedule:$BUILD_NUMBER
                       '''
                    }
                }
            }
        }
        stage('CanaryDeploy') {
       //     when {
       //         branch 'master'
       //     }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
       //     when {
       //         branch 'master'
       //     }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
