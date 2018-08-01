#!groovy

pipeline {
      agent any
   // agent {
   //     label 'perf'
   // }
    //triggers {
        // On any repo change.
        //pollSCM('H/3 * * * *')
    //}
    options {
        skipDefaultCheckout()
    }
    environment {
        NOTIFY_SLACK_CHANNEL = '#team-infracloud'
        REPO_NAME = 'github.com/ShiftLeftSecurity/infracloud-perf'
        GITHUB_KEY = '4b3482c3-735f-4c31-8d1b-d8d3bd889348'
        FREQUENCY = 5
        DURATION = 120
    }
    stages {
      stage('Git Pull') {
        steps {
          git branch: 'master', credentialsId: '89b56f52-38a5-40c7-93af-4c788b3ee76c', url: 'https://github.com:ipochi/hsl-prometheus-cadvisor-grafana.git'
          // script {
          //   sshagent (credentials: ["${env.GITHUB_KEY}"]) {
          //     checkout([$class: 'GitSCM', branches: [[name: "*/jenkinsfile-changes"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: "${env.GITHUB_KEY}", url: "ssh://git@${env.REPO_NAME}"]]])
          //   }
          // }
        }
      }
      stage('Docker pull') {
        steps {
          // add steps here to pull docker images
          sh 'docker pull infracloud/hsl-withsl:v1'
          sh 'docker pull infracloud/vloadgenerator:v1'
        }
      } 
  
      stage("Start docker compose for monitoring ...") {
        steps {
          sh 'docker-compose -f docker-compose/docker-compose-monitor.yaml up'
            // Sleeping for 1 min to start prometheus , grafana and cadvisor
            sh 'sleep 60'
            sh './script/init.sh admin:admin'
        }
      }
      
      stage("Start docker containers for test") {
        steps  {
          sh 'FREQUENCY=${env.FREQUENCY} DURATION=${env.DURATION} docker-compose -f docker-compose/docker-compose-hsl-withsl.yaml up --abort-on-container-exit'
        }
      }

      stage("Stop monitoring after test finishes") {
        steps {
          sh 'docker-compose -f docker-compose/docker-compose-monitor.yaml down'
        }
      }
    }

    // post {
    //     failure {
    //         script {
    //             notifyFailed()
    //         }
    //     }
    //     aborted {
    //         script {
    //             notifyAborted()
    //         }
    //     }
    //     fixed {
    //         script {
    //             notifySuccess()
    //         }
    //     }
    // }
}

def notifyAborted() {
    slackSend (channel: "${env.NOTIFY_SLACK_CHANNEL} ",
            color: '#777777',
            message: "ABORTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}

def notifyFailed() {
    slackSend (channel: "${env.NOTIFY_SLACK_CHANNEL} ",
            color: '#FF0000',
            message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}

def notifySuccess() {
    slackSend(channel: "${env.NOTIFY_SLACK_CHANNEL} ",
            color: '#22FF00',
            message: "FIXED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
