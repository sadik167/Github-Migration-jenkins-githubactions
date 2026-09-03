pipeline {
    agent any

    environment{
        AWS_REGION = "us-east-1"
        ECR_REPOSITORY ="jenkins-migration-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_PORT = "8090"
    }
    stages {
        stage('git checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/sadik167/Github-Migration-jenkins-githubactions'
            }
        }
		
		 stage('TEST') {
            steps {
               sh '''
                   echo "Running application tests ..."
                   cd app
                   npm install
                   npm test
                 '''
            }
        }
		
		stage('AWS CRENTIALS') {
            steps {
			  script{
               env.AWS_ACCOUNT_ID = sh(
			   script: 'aws sts get-caller-identity --query Account --output text',
			   returnStdout: true).trim()
			   env.AWSECR_REGISTRY = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com"
            }
			
			sh '''
			    echo "AWS authentication successfull"
				aws sts get-caller-identity
				
			   '''
			}
        }
		
		stage('Build Docker Image') {
            steps {
			  sh '''
			      docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} .
				  docker image ls | grep ${ECR_REPOSITORY}
				 '''
        }
		
	}
	
	stage('Login to ECR') {
            steps {
			  sh '''
			      aws ecr get-login-password --region ${AWS_REGION} | \
				  docker login --username AWS --password-stdin ${AWSECR_REGISTRY}
				  docker image ls | grep ${ECR_REPOSITORY}
				 '''
        }
		
	}
	
	stage('PUSH IMAGE to ECR') {
            steps {
			  sh '''
			      docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${AWSECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
				  docker push ${AWSECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
				  
				 '''
        }
		
	}
	
	stage('Deploy') {
            steps {
			  sh '''
			      docker stop jenkins-migration || true
				  docker rm jenkins-migration || true
				  
				  docker pull ${AWSECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
				  
				  docker run -d --name jenkins-migration -p ${APP_PORT}:8080 ${AWSECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
				  
				  docker ps -a
				  
				 '''
        }
		
	}
	
	stage('Smoke Test') {
            steps {
			  sh '''
			      sleep 5
				  curl --fail http://localhost:${APP_PORT}/health
				  echo "smoke test passed successfully!"
				  
				 '''
        }
		
	}
	
	
}
post{
	       success { echo "Jenkins pipeline completed successfully"}
		   failure{ echo "Jenkins pipleine failed "}
		   
		   always{
		   echo "Build Number: ${BUILD_NUMBER}"
		   echo "Image Tag: ${IMAGE_TAG}"
		   
		   }
	
	}

}