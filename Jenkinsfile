try {
	node {
		stage('Configure Tools') {
			// Add Node (including NPM) to the path
			def nodeHome = tool 'Node-v6.9.4LTS'
			env.PATH = "${env.PATH}"
		}

		stage('Checkout Source Code') {
			// Checkout source code from Git
			checkout([
				$class: 'GitSCM',
				branches: [[name: '*/master']], // master branch
				doGenerateSubmoduleConfigurations: false,
				extensions: [],
				submoduleCfg: [],
				userRemoteConfigs: [[credentialsId: 'Bitbucket', url: 'http://jenkins@v187w08r2vxnc:7990/scm/prod/vitic.git']] // vitic repository
			])
		}
		
		stage('Backend Build') {
			// Run the maven build
			bat '''
				cd org.lhasalimited.vitic/org.lhasalimited.vitic.backend.web
				mvn clean install
			'''
		}
		
		stage('Deploy Backend') {
			// copy jar file and run windows service
			bat '''
				cd org.lhasalimited.vitic\\org.lhasalimited.vitic.backend.web
				echo "Stop and waiting for backend service to stop"
				call src\\main\\resources\\windows.service\\StopService.bat
				echo "Deploying backend"
				call copy target\\org.lhasalimited.vitic.backend.web-0.0.1-SNAPSHOT.jar "\\\\v187w08r264nc\\Vitic\\org.lhasalimited.vitic.backend.web.jar"
				call copy src\\main\\resources\\windows.service\\ViticBackend.xml "\\\\v187w08r264nc\\Vitic\\"
				call copy src\\main\\resources\\windows.service\\ViticBackend.exe "\\\\v187w08r264nc\\Vitic\\"
				echo "Starting backend service"
				call sc \\\\v187w08r264nc start viticbackend
			'''
		}
		
		stage('Frontend Build') {
			// Run the frontend tests
			bat '''
				echo "going to start the npm tests"
				echo "version of node is:"
				node -v
				echo "version of npm is:"
				call npm -v
				echo "changing directory to the web frontend directory to do the build from there"
				cd org.lhasalimited.vitic/org.lhasalimited.vitic.frontend.web
				echo "installing node modules"
				call npm i
				echo "running node tests"
				call npm run test-jenkins
				echo "building dev dist"
				call npm run build:dev
				echo "running e2e tests"
				call npm run e2e
			'''
		}

		stage('Deploy Frontend'){
			// Run npm build
			bat '''
					cd org.lhasalimited.vitic/org.lhasalimited.vitic.frontend.web
					echo "Build production artifacts"
					call npm run build:prod
					echo "Deleting vitic folder"
					call rmdir "\\\\v187w08r264nc\\Apache24\\htdocs\\vitic" /S /Q
					echo "make new vitic folder"
					mkdir "\\\\v187w08r264nc\\Apache24\\htdocs\\vitic"
					echo "copy new generated distribution"
					call xcopy /s /Y dist "\\\\v187w08r264nc\\Apache24\\htdocs\\vitic"
				'''
		}
	
		stage('API Tests'){
			// Run the maven build
			bat '''
					cd org.lhasalimited.vitic/org.lhasalimited.vitic.api.test
					mvn clean install -P!noTest -Dtest=ApiTestRunner -Durl=http://v187w08r264nc:9090
				'''
		}

		stage('Acceptance Tests'){
			// Run mvn test
			bat '''
					cd org.lhasalimited.vitic/org.lhasalimited.vitic.acceptance.test
					mvn clean install -P!noTest -Dtest=AcceptanceTestRunner -Durl=http://v187w08r264nc//vitic
				'''
		}
		
	}
} catch (error) {
	def emailTo = 'paul.sehgal@lhasalimited.org ian.addison@lhasalimited.org richard.stevenson@lhasalimited.org fiona.mcDonald@lhasalimited.org mohammed.sultan@lhasalimited.org'
		mail body: "Vitic pipeline job failed, please see $JOB_URL" ,
			from: 'jenkins@lhasalimited.org',
			subject: 'Vitic Pipeline Job Failed',
			to: emailTo
	throw error
}
