#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    docker.image('lcastrooliveira/java-libxss1').inside('-u root -e MAVEN_OPTS="-Duser.home=./"') {
        stage('check java') {
            sh "java -version"
        }

        stage('clean') {
            sh "chmod +x mvnw"
            sh "./mvnw clean"
        }

        stage('install tools') {
            sh "./mvnw com.github.eirslett:frontend-maven-plugin:install-node-and-yarn -DnodeVersion=v6.11.3 -DyarnVersion=v1.1.0"
        }

        stage('yarn install') {
            sh "./mvnw com.github.eirslett:frontend-maven-plugin:yarn"
        }

        stage('backend tests') {
            try {
                sh "./mvnw test"
            } catch(err) {
                throw err
            } finally {
                junit '**/target/surefire-reports/TEST-*.xml'
            }
        }

        stage('frontend tests') {
            try {
                sh "./mvnw -e com.github.eirslett:frontend-maven-plugin:gulp -Dfrontend.gulp.arguments=test"
            } catch(err) {
                throw err
            } finally {
                junit '**/target/test-results/karma/TESTS-*.xml'
            }
        }

        stage('packaging') {
            sh "./mvnw package -Pprod -DskipTests"
            archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
        }

        stage('quality analysis') {
            withSonarQubeEnv('sonarqube') {
                sh "./mvnw sonar:sonar"
            }
        }
    }

    def dockerImage
    stage('build docker') {
        sh "cp -R src/main/docker target/"
        sh "cp target/*.war target/docker/"
        dockerImage = docker.build('lasse/jhipstersampleapplication', 'target/docker')
    }

    stage('publish docker') {
        docker.withRegistry('179.106.217.217:5000') {
            dockerImage.push 'latest'
        }
    }
}
