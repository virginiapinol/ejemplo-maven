import groovy.json.JsonOutput

def COLOR_MAP =[
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

def getBuildUser() {
  def userCause = currentBuild.rawBuild.getCause(Cause.UserIdCause)
  def upstreamCause = currentBuild.rawBuild.getCause(Cause.UpstreamCause)

  if (userCause) {
    return userCause.getUserId()
  } else if (upstreamCause) {
    def upstreamJob = Jenkins.getInstance().getItemByFullName(upstreamCause.getUpstreamProject(), hudson.model.Job.class)
    if (upstreamJob) {
      def upstreamBuild = upstreamJob.getBuildByNumber(upstreamCause.getUpstreamBuild())
      if (upstreamBuild) {
        def realUpstreamCause = upstreamBuild.getCause(Cause.UserIdCause)
        if (realUpstreamCause) {
          return realUpstreamCause.getUserId()
        }
      }
    }
  }
}

pipeline {
    agent any
    environment{
        BUILD_USER = ''
    }
    stages {
        stage('Compilación') {
            steps {
                sh './mvnw clean compile -e'
            }
        }
        stage('Test') {
            steps {
                sh './mvnw clean test -e'
            }
        }
        stage('Análisis Sonarqube') {
            environment {
                scannerHome = tool 'SonarScanner'
            }
            steps {
                 withSonarQubeEnv('sonar') {
                    sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=ejemplo_maven -Dsonar.host.url=https://1af485c76019.sa.ngrok.io -Dsonar.login=sqp_698c2fe99ed14e165a65f6d9ca088a8edc9af442'
                }
            }
            
        }
        stage("Comprobación Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        } 
        stage('Jar Code') {
            steps {
                sh './mvnw clean package -e'
            }
        }
        stage('Run Jar') {
            steps {
                sh 'nohup bash mvnw spring-boot:run &'
            }
        }
    }
    post{
        success{
            setBuildStatus("Build succeeded", "SUCCESS");
        }

        failure {
            setBuildStatus("Build failed", "FAILURE");
        }
    }
}

void setBuildStatus(String message, String state) {
    step([
        $class: "GitHubCommitStatusSetter",
        reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/virginiapinol/ejemplo-maven"],
        contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
        errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
        statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]]]
    ]);
}
