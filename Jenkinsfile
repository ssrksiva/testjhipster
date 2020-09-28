#!/usr/bin/env groovy

node {
    def NODE_HOME = tool 'Node'
	def DOCKER_HOME = tool 'docker'
	env.PATH="${NODE_HOME}/bin/:${DOCKER_HOME}:${env.PATH}"
	
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        sh "java -version"
    }

    stage('clean') {
        sh "chmod +x mvnw"
        sh "./mvnw -ntp clean -P-webpack"
    }
    stage('nohttp') {
        sh "./mvnw -ntp checkstyle:check"
    }

	 
    stage('install tools') {
	withNPM(npmrcConfig:'MyNpmrcConfig') {
            echo "Performing npm build..."
            sh 'npm install'
        }
    }
	
    stage('backend tests') {
        try {
            sh "./mvnw -ntp verify -P-webpack"
        } catch(err) {
            throw err
        } finally {
            junit '**/target/test-results/**/TEST-*.xml'
        }
    }

    stage('frontend tests') {
        try {
            sh "./npm run test"
        } catch(err) {
            throw err
        } finally {
            junit '**/target/test-results/**/TEST-*.xml'
        }
    }

    stage('packaging') {
        sh "./mvnw -ntp verify -P-webpack -Pprod -DskipTests"
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }

    def dockerImage
    stage('publish docker') {
        // A pre-requisite to this step is to setup authentication to the docker registry
        // https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin#authentication-methods
        sh "./mvnw -ntp jib:build"
    }
}
