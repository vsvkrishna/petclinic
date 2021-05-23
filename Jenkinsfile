pipeline {
    agent any
    tools {
    	maven 'M3.8'
    }

    stages {
    	stage('Build') {
    		steps {
				sh "mvn clean package"
			}
		}	

		stage('Post Build Actions') {
			parallel {
				stage('Nexus Upload - Release') {
					steps {
						nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '', file: 'target/petclinic.war', type: 'war']], credentialsId: 'nexusCred', groupId: 'org.springframework.samples', nexusUrl: '3.138.188.192:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-releases', version: '4.2.6'
					}
				}

				stage('Archive artifacts') {
					steps {
						archiveArtifacts artifacts: "target/*.?ar", followSymlinks: false
					}
				}

				stage('Junit') {
					steps {
						junit "target/surefire-reports/*.xml"
					}
				}
			}
		}
    }
    post {
    	success {
    		notify('Success')
    	}
    	failure {
    		notify('Failed')
    	}
    }
}

def notify(status) {
	emailext (
		body: """
			<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        	<p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>
			""", 
		subject: "${status}: ${env.JOB_NAME} - [${env.BUILD_NUMBER}]", 
		to: 'devops.kphb@gmail.com'
	)
}
