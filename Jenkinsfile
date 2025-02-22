pipeline {

    agent {
        label 'My-Jenkins-Agent'
    }
    // agent any

    environment {
        APP_NAME = "devops-003-pipeline-aws"
        RELEASE = "1.0"
        DOCKER_USER = "asoner01"
        DOCKER_LOGIN = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}.${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials ("JENKINS_API_TOKEN")
    }
    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                //   checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/AbdullahSalihOner/DevOps-03-Pipeline-Aws']])
                git branch: 'master', credentialsId: 'github', url: 'https://github.com/AbdullahSalihOner/DevOps-03-Pipeline-Aws'
            }
        }
        stage('Build Maven') {
            steps {
                //  sh 'mvn clean install'
                //  bat 'mvn clean install'

               sh 'mvn clean package'
               // bat 'mvn clean package'

            }
        }


        stage('Test Application') {
            steps {
                sh 'mvn test'
                //  bat 'mvn test'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }



/*
       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
*/

        stage('Build & Push Docker Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_LOGIN) {
                        docker_image = docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }


        stage("Trivy Scan") {
            steps {
                script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image asoner01/devops-03-pipeline-aws:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }


        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"


                    // Agent makinesi zamanla dolacak. Docker şişecek dolacak. Temizlik yapmanız lazım.
                    // Agent makinede temizlik için yeriniz azalmışsa şu komutları kulanın lütfen.
                    // Hatta mümkünse bu kodları buraya uyarlayın lütfen.
                    /*
                    docker rmi $(docker images --format '{{.Repository}}:{{.Tag}}' | grep 'devops-03-pipeline-aws')

                    docker container rm -f $(docker container ls -aq)

                    docker volume prune
                    */

                }
            }
        }



     stage("Trigger CD Pipeline") {
            steps {
                script {
                      sh "curl -v -k --user salihoner:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-44-197-242-168.compute-1.amazonaws.com:8080/job/devops-003-pipeline-aws-gitops/buildWithParameters?token=GITOPS_TOKEN_INFO'"
                  }
            }
       }



    }
}