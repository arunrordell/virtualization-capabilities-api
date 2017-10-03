UPSTREAM_TRIGGERS = [
    "common-dependencies",
    "common-messaging-parent"
]
properties(getBuildProperties(upstreamRepos: UPSTREAM_TRIGGERS))

pipeline {
    parameters {
        choice(choices: 'OFF\nON', description: 'Please select appropriate flag (master and stable branches will always be ON)', name: 'Deploy_Stage')
    }
    triggers {
        upstream(upstreamProjects: UPSTREAM_TRIGGERS, threshold: hudson.model.Result.SUCCESS)
    }
    agent {
        node {
            label 'maven-builder'
            customWorkspace "workspace/${env.JOB_NAME}"
        }
    }
    environment {
        GITHUB_TOKEN = credentials('github-02')
    }
    options {
        skipDefaultCheckout()
        timestamps()
    }
    tools {
        maven 'linux-maven-3.3.9'
        jdk 'linux-jdk1.8.0_102'
    }
    stages {
        stage('Checkout') {
            steps {
                doCheckout()
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean install -Dmaven.repo.local=.repo"
            }
        }
        stage('Record Test Results') {
            steps {
                junit '**/target/*-reports/*.xml'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                doSonarAnalysis()
            }
        }
        stage('Third Party Audit') {
            steps {
                doThirdPartyAudit()
            }
        }
        stage('PasswordScan') {
            steps {
                doPwScan()
            }
        }
        stage('Deploy') {
            steps {
                doMvnDeploy()
            }
        }
        stage('Github Release') {
            steps {
                githubRelease()
            }
        }
        stage('NexB Scan') {
            steps {
                doNexbScanning()
            }
        }
    }
    post {
        always {
            cleanWorkspace()
        }
        success {
            successEmail()
        }
        failure {
            failureEmail()
        }
    }
}
