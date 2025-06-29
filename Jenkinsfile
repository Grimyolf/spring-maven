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
		            // Deploy the WAR file to Nexus Repository
		            nexusArtifactUploader(
		                nexusVersion: 'nexus3',
		                protocol: 'http',
		                serverId: '192.168.138.132:8081/',
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

    }

}
