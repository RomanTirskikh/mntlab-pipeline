node (env.SLAVE) {
    env.student='vulantsau'
    //customWorkspace '/opt/jenkins/master/workspace/customWorkspace'
    stage('Preparation(Checking out)') {
        checkout([$class: 'GitSCM', branches: [[name: '*/vulantsau']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/MNT-Lab/mntlab-pipeline.git']]])
        }
    stage('Building code') {
        sh "gradle build"
    }
    stage ('Testing code') {
        parallel( 
            Unittests: {
                stage ('Unit Tests') {
                    sh "gradle test"
                }
            },
            jacocoTests: {
                stage ('Jacoco Tests') {
                    sh "gradle jacocoTestReport"
                    }
            },
            cucumberTests: {
                stage ('Cucumber Tests') {
                    sh "gradle cucumber"
                    }
            }
        )
    }
    stage('Triggering job and fetching artifact after finishing') {
        build job: 'MNTLAB-vulantsau-child1-build-job', parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: 'vulantsau']]
        step([$class: 'CopyArtifact', filter: 'vulantsau_dsl_script.tar.gz	', fingerprintArtifacts: true, flatten: true, projectName: 'MNTLAB-vulantsau-child1-build-job', target: ''])
    }
    stage ('Packaging and Publishing results') {
        /*a)*/ sh 'tar xvf vulantsau_dsl_script.tar.gz'
        /*b)*/ sh 'tar zvfc pipeline-${student}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile build/libs/'+JOB_NAME.replace(env.SLAVE+'/',"")+'.jar'
        /*c)*/ archiveArtifacts artifacts: 'pipeline-'+student+'-${BUILD_NUMBER}.tar.gz', allowEmptyArchive: false
        /*d)*/ sh 'curl -v -u admin:admin123 --upload-file pipeline-'+student+'-${BUILD_NUMBER}.tar.gz http://10.6.102.46:8081/repository/artifact/pipeline'+student+'-${BUILD_NUMBER}.tar.gz'
        /*d)*/ //nexusArtifactUploader artifacts: [[artifactId: 'task11ArtifactId', classifier: '', file: 'pipeline-'+env.student+'-${BUILD_NUMBER}.tar.gz' , type: 'tar.gz']], credentialsId: 'admin', groupId: 'task11GroupId', nexusUrl: '10.6.103.32:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'artifacts', version: '1.0'
    }
    stage ('Asking for manual approval') {
        timeout(time:5, unit:'DAYS') {
            input 'Approve deployment?' //, submitter: 'student'
        }
    }
    stage('Deployment') {

        sh 'chmod -R 766 build/libs/'
        if(!(sh(script: 'java -jar build/libs/'+JOB_NAME.replace(env.SLAVE+'/',"")+'.jar', returnStdout: true)).contains("Hello World!")) {
            currentBuild.result = 'FAILURE'
        }
    }
    stage ('Sending status') {
        echo 'SUCCESS'
    }
}
