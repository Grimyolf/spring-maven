pipeline {
    	agent any
    	tools {
       		maven 'localmaven'
        	jdk "java17"
    	}
    	environment {
        	PROJECT_VERSION = '0.0.1'
        	ARTIFACT_ID = 'demo-github-thomasd'
        	GROUP_ID = 'com.example'
        	DOCKER_IMAGE = 'grimyolf/demo-github-thomasd'
        	DOCKERHUB_CREDENTIALS = 'dockerhub'
    	}
    	stages {
		stage('Checkout') {
		    steps {
		        script {
		            // Checkout your source code from version control
		            git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/Grimyolf/spring-maven.git';
		        }
		    }
		}

		stage('Build') {
		    steps {
		        script {
		            // Build the Maven project
		            sh "mvn clean package"
		        }
		    }
		}
		stage('SonarQube Analysis') {
		    steps {
		        withSonarQubeEnv('SonarQube') {
		            sh 'mvn sonar:sonar -Dsonar.projectKey=demo-github-thomasd -Dsonar.projectName=DemoThomas -Dsonar.host.url=http://localhost:9000'
		        }
		    }
		}
		stage('Deploy to Nexus') {
		    steps {
		        script {
		            nexusArtifactUploader(
		                nexusVersion: 'nexus3',
		                protocol: 'http',
		                serverId: 'nexus',
		                groupId: "${GROUP_ID}",
		                version: "${PROJECT_VERSION}",
		                repository: 'AppSpring',
		                credentialsId: 'nexus-creds',
		                nexusUrl: '127.0.0.1:8081',
		                artifacts: [
		                    [artifactId: "${ARTIFACT_ID}",
		                     classifier: '',
		                     file: "target/${ARTIFACT_ID}-${PROJECT_VERSION}.jar",
		                     type: 'jar']
		                ]
		            )
                	}
            		}
        	}
        	stage('Build Docker Image') {
            		steps {
                		script {
                    			sh "docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} ."
                		}
            		}
        	}

        	stage('Push Docker Image') {
            		steps {
                		script {
                    			docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        		sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                    			}
                		}
            		}
        	}
        	stage('Deploy Docker Container') {
    			steps {
        			script {
				    sh '''
					echo ">>> Nettoyage de l'ancien container (s'il existe)"
					docker stop app-thomasd-secops || true
					docker rm app-thomasd-secops || true

					echo ">>> Lancement du nouveau container"
					docker run -d --name app-thomasd-secops -p 8037:8080 grimyolf/demo-github-thomasd:9

					echo ">>> Déploiement terminé, application disponible sur le port 8037"
				    '''
        			}
    			}
		}
		stage('Scan Docker Image with Trivy') {
    			steps {
        			script {
            				sh 'trivy image grimyolf/demo-github-thomasd:9'
        			}
    			}
		}

    	}

}
