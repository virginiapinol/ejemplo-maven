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
                    sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=ejemplo_maven -Dsonar.host.url=https://1af485c76019.sa.ngrok.io -Dsonar.login=squ_6c8cafa0da0ac1956dc6a6f29d83e7d01856b654 -Dsonar.target=sonar.java.binaries'
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
        stage ('Publish Nexus'){
            steps{
                nexusPublisher nexusInstanceId: 'Nexus-1', nexusRepositoryId: 'devops-usach-vpino', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: './build/DevOpsUsach2020-0.0.1.jar']], mavenCoordinate: [artifactId: 'DevOpsUsach2020', groupId: 'com.devopsusach2020', packaging: 'jar', version: '0.0.1']]]
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
