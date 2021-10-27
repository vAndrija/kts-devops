#!/usr/bin/env groovy

pipeline {
    environment {
        registry = 'backend-testapp'
        dockerImage = ''
        build_sha = ''
        job = 'test-job'
        development_app = 'testapp-andros-develop'
    }

    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '14', daysToKeepStr: '', numToKeepStr: '14'))
    }

    parameters {
        string(name: 'Branch', defaultValue: 'develop', description: 'Which branch are we building?', trim: true)
        
    }

    stages {
        stage('parameterizing') {
            when {
                expression { params.Invoke_Parameters == true }
            }
            steps {
                script {
                    currentBuild.result = 'ABORTED'
                    error('DRY RUN COMPLETED. JOB PARAMETERIZED.')
                }
            }
        }
        stage('Pulling App Code') {
            steps {
                git  url: 'https://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@github.com/vAndrija/kts-backend', branch: params.Branch, changelog: 'True'
                script { build_sha = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim() }
            }
        }
        stage('Pushing Containers: Develop') {
            when {
                expression {params.Branch == "master"}
            }
            steps{
                sh "heroku container:push web --app ${development_app}"
                sh "heroku container:release web --app ${development_app}"
            }
        }
        stage('Pushing Containers: Master') {
            when {
                expression {params.Branch == "main"}
            }
            steps{
                sh "heroku container:push web --app ${production_app}"
                sh "heroku container:release web --app ${production_app}"
            }
        }
        stage('Cleaning up build artifacts: Develop') {
            steps {
                sh 'ls -alF'
                sh 'docker images'
                sh "docker rmi registry.heroku.com/${development_app}/web"
                sh "docker rmi --force \$(docker images -f \"dangling=true\" -q)"
                sh 'ls -alF'
                sh 'docker images'
            }
        }
        stage('Cleaning up build artifacts: Master') {
            steps {
                sh 'ls -alF'
                sh 'docker images'
                sh "docker rmi registry.heroku.com/${production_app}/web"
                sh "docker rmi --force \$(docker images -f \"dangling=true\" -q)"
                sh 'ls -alF'
                sh 'docker images'
            }
        }
    }
}
