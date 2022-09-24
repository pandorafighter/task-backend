pipeline {
	agent any
	stages 
    {
		stage ('Build Backend') {
			steps {
				bat 'mvn clean package -DskipTests=true'
            }
		}
		stage ('Unit Tests') {
			steps {
				bat 'mvn clean test verify'
			}
		}
		stage ('Sonar Analysis') {
			environment {
				scannerHome = tool 'SONAR_SCANNER'
			}
			steps {
				withSonarQubeEnv('SONAR_REMOTE') {
					bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://192.168.5.135:9000/ -Dsonar.login=sqp_30ecc0118962a9fa3a4a6ecb99d08b851f2ffc5c -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java,**RootController.java"
				}
			}
		}
		stage ('QualityGate') {
			steps {
				sleep(5)
				timeout(time:1, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
		stage ('Deploy Backend') {
			steps {
				deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://192.168.5.135:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
			}
		}
		stage ('API Test') {
			steps {
				dir('api-test') {
					git credentialsId: 'RepoPandora', url: 'https://github.com/pandorafighter/tasks-api-test'
					bat 'mvn test'
				}
			}
		}
		stage ('Deploy Frontend') {
			steps {
				dir ('frontend') {
					git credentialsId: 'RepoPandora', url: 'https://github.com/pandorafighter/tasks-fronend'
					bat 'mvn clean package'
					deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://192.168.5.135:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
				}				
			}
		}
		stage ('Fuctional Test') {
			steps {
				dir('functional-test') {
					git credentialsId: 'RepoPandora', url: 'https://github.com/pandorafighter/tasks-functional-tests'
					bat 'mvn test'
				}
			}
		}
	}
	    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, fuctional-test/target/surefire-reports/*.xml, fuctional-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war,frontend/target/tasks.war', caseSensitive: false, defaultExcludes: false, followSymlinks: false, onlyIfSuccessful: true
        }
    }
}