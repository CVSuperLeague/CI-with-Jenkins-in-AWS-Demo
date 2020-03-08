pipeline {
    agent any
    environment {
        PROJECT_ID = 'devops-cv-tcssuper'
        CLUSTER_NAME = 'kubernetes-cluster'
        LOCATION = 'us-central1-a'
        CREDENTIALS_ID = 'kubernetes'
    }
    stages {
        stage("Code checkout") {
            steps {
                checkout scm
            }
        }
		 stage("Build") {
            steps {
               echo "cleaning and packaging"
			   sh 'mvn clean package'
            }
        }
		 stage("Test") {
            steps {
                echo "Testing"
			   sh 'mvn test'
            }
        }
        stage("Build gcr image") {
            steps {
                script {
                    myimage = docker.build("gcr.io/devops-cv-tcssuper/gcr-cvdevops/devops:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://gcr.io', 'gcr:gcr_credentials') {
                            myimage.push("${env.BUILD_ID}")
                    }
                }
            }
        }        
        stage('Deploy to K8s') {
            steps{
			    echo "Deployment started"
				sh 'ls -ltr'
				sh 'pwd'
                sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
				echo "Deployment Finished"
            }
        }
    }    
}
