node(env.SLAVE) {
	//env.PATH=env.PATH+":/opt/gradle-4.0/bin:/opt/groovy/groovy-2.3.0-beta-2/bin"
	stage('Preparation (Checking out)') {
		git branch: 'rtirskikh', url: 'https://github.com/RomanTirskikh/mntlab-pipeline.git'
	}

	stage ('Building code') {
        	sh "gradle build"
	}

 	stage ('Testing code'){
       	parallel(
			'Unit Tests':{
				sh 'gradle cucumber'
			},

			'Jacoco Tests':{
				sh 'gradle jacocoTestReport'
			},
			'Cucumber Tests':{
				sh 'gradle test'

			}
		)
	}

  	stage ' Triggering job and fetching artifacts'

	build job: 'EPMFARMDVO300-MNTLAB-rtirskikh-child1-build-job', parameters: [string(name: 'BRANCH_NAME', value: 'rtirskikh')], wait: true
	step([$class: 'CopyArtifact', projectName: "EPMFARMDVO300-MNTLAB-rtirskikh-child1-build-job", filter: '*.tar.gz']);
	wrap([$class: 'TimestamperBuildWrapper']) {
     	echo "Job was triggered and finished"
   }

   	stage ('Packaging and Publishing results'){
		sh 'tar -xf rtirskikh_dsl_script.tar.gz'
		sh 'tar -czf rtirskikh-"${BUILD_NUMBER}".tar.gz jobs.groovy Jenkinsfile -C build/libs/ gradle-simple.jar'
		//nexusArtifactUploader artifacts: [[artifactId: "${BUILD_NUMBER}", classifier: 'tar.gz', file: '/target/pipeline-rtirskikh-"${BUILD_NUMBER}".tar.gz', type: "${BUILD_NUMBER}"]], credentialsId: 'admin', groupId: 'groupid', nexusUrl: '192.168.56.51:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'artifact', version: 'release'
		sh 'groovy upload_download.groovy upload rtirskikh-${BUILD_NUMBER}.tar.gz'
	}

	stage ('Asking for manual approval'){
		input 'Deploy or Abort?'
	}

	stage ('Deployment'){
		sh 'java -jar build/libs/gradle-simple.jar'
	}

	stage ('Sending status'){
		echo 'SUCCESS'
	}
}
