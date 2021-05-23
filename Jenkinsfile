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
