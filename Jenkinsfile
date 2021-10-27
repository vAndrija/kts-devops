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
        string(name: 'Branch', defaultValue: 'master', description: 'Which branch are we building?', trim: true)
        choice(name: 'DeployTarget', choices: ['no_deploy', 'dev', 'prod'], description: 'Which environment are we deploying this to?')
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
                git  url: 'https://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@github.com/andros3/learning', branch: params.Branch, changelog: 'True'
                script { build_sha = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim() }
            }
        }
        stage('Printing command'){
            steps{
                sh "echo ${params.Branch}"
            }
        }
        stage('Pushing Containers: Master') {
            when {
                expression {params.Branch == "master"}
            }
            steps{
                sh "heroku container:push web --app ${development_app}"
                sh "heroku container:release web --app ${development_app}"
            }
        }
        stage('Pushing Containers: Main') {
            when {
                expression {params.Branch == "main"}
            }
            steps{
                sh "heroku container:push web --app ${development_app}"
                sh "heroku container:release web --app ${development_app}"
            }
        }
        stage('Cleaning up build artifacts') {
            steps {
                sh 'ls -alF'
                sh 'docker images'
                sh "docker rmi registry.heroku.com/${development_app}/web"
                sh "docker rmi --force \$(docker images -f \"dangling=true\" -q)"
                sh 'ls -alF'
                sh 'docker images'
            }
        }
    }
}
