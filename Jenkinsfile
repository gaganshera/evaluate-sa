pipeline{
    environment {
    username = 'samraazeem'
    dockerPort = "${env.BRANCH_NAME == "develop" ? 7300 : 7200}"
    dockerUsername = "samraazeem"
    registry = "samraazeem/"
    registryCredential = 'docker'
    dockerImage= ''
    kubernetesContext = "Istio-Cluster"
    }
    agent any

    tools {
        nodejs "nodejs"
        dockerTool 'docker'
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage("Code Checkout") {
            steps {
                git url: 'https://github.com/samraazeem/app_samraazeem.git'
            }
        }

        stage('Build') {
            steps{
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Unit Testing'){
            when {
                branch 'master-original'
            }
            steps{
               sh 'ng test --codeCoverage=true --watcher=true'
            }
        }

        stage('SonarQube Analysis'){
            when {
                branch 'production'
            }
			steps{
				withSonarQubeEnv('Test_Sonar'){
					sh 'npm run sonar'
				}
			}
		}

        stage('Docker Image') {
            steps{
                script {
                    dockerImage= docker.build registry + "i-${username}-${BRANCH_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Container'){
            parallel {
                stage('PreContainer Check'){
                    when {
                        expression {
                            return sh (script: "docker port c-${username}-${BRANCH_NAME}", returnStatus: true) == 0;
                        }
                    }
                    steps{
                        sh(script: "docker stop c-${username}-${BRANCH_NAME}", returnStatus: true)
                        sh(script: "docker rm -f c-${username}-${BRANCH_NAME}", returnStatus: true)
                    }
                }
                stage('Publish DockerHub'){
                    steps{
                        sh "docker tag i-${username}-${BRANCH_NAME}:${BUILD_NUMBER} ${dockerUsername}/i-${username}-${BRANCH_NAME}:${BUILD_NUMBER}"
                        sh "docker tag i-${username}-${BRANCH_NAME}:${BUILD_NUMBER} ${dockerUsername}/i-${username}-${BRANCH_NAME}"
                        sh "docker push ${dockerUsername}/i-${username}-${BRANCH_NAME}:${BUILD_NUMBER}"
                        sh "docker push ${dockerUsername}/i-${username}-${BRANCH_NAME}"
                    }
                }
            }
        }
        stage('Docker Deployment'){
            steps{
                sh 'docker run --name c-${username}-${BRANCH_NAME} -d -p ${dockerPort}:80 -e branch=${BRANCH_NAME} i-${username}-${BRANCH_NAME}:${BUILD_NUMBER}'
            }
        }
        stage('Kubernetes Deployment'){
            steps{
                //sh 'kubectl apply -f ./kubernetes/frontend.yaml -n=kubernetes-cluster-samraazeem'
                //sh 'kubectl apply -f ./kubernetes/backend.yaml -n=kubernetes-cluster-samraazeem'
                sh 'npm install'
            }
        }
    }
}
