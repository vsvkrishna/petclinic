pipeline {
	agent any
	tools {
		maven 'M3.6'
	}
	stages {
		stage('Build') {
			steps {
    			sh "mvn clean package"
    		}
		}
		stage('Post Build Actions') {
			parallel {
				stage('Test Result') {
					steps {
						junit "target/surefire-reports/*.xml"
					}
				}
				stage('Archive Artifact') {
					steps {
					    archiveArtifacts artifacts: "target/*.?ar", followSymlinks: false
					}
				}
				stage('Nexus upload') {
					steps {
						nexusArtifactUploader artifacts: [
						[artifactId: 'spring-petclinic', 
						classifier: '', 
						file: "target/petclinic.war", 
						type: 'war']], 
						credentialsId: 'nexusCred', 
						groupId: 'org.springframework.samples', 
						nexusUrl: '18.222.110.235:8081/nexus', 
						nexusVersion: 'nexus2', 
						protocol: 'http', 
						repository: 'releases', 
						version: "4.2.${BUILD_NUMBER}"
					}
				}
			}
		}
		stage('Deploy') {
			input {
                		message "Should we continue?"
                		ok "Yes, we should."
            		}
			steps {
				// git 'https://github.com/akmaharshi/tomcat-standalone.git'
				checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, 
                                          extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'ansible']], submoduleCfg: [], 
        				userRemoteConfigs: [[url: 'https://github.com/akmaharshi/tomcat-standalone.git']]])
				
				withCredentials([string(credentialsId: 'ansi_vault_pass', variable: 'MYPASS')]) {
					sh '''
						echo $MYPASS
						echo $MYPASS > ~/.vault_pass.txt
						export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt
						cd ansible
						sudo ansible-playbook -i production -e "BUILD_NO=${BUILD_NUMBER}" --vault-id ~/.vault_pass.txt site.yml 
					'''
				}
			}
		}
	}
	post {
		success {
			echo "Success"
		}
		failure {
			echo "Failed"
		}
	}
}
