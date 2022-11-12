pipeline {
    agent any
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
      stage('SCM') {
        git 'https://github.com/virginiapinol/ejemplo-maven.git'
      }
      stage('SonarQube analysis') {
        def scannerHome = tool 'SonarScanner 4.0';
        withSonarQubeEnv('sonarVirginia') { // If you have configured more than one global server connection, you can specify its name
          sh "${scannerHome}/bin/sonar-scanner"
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

def gitmerge(String Originbranch, String destinybranch) {
    println "Realizando checkout" ${Originbranch} y ${destinybranch}
    
    checkout(Originbranch)
    checkout(ramaDestino)
    
    println "Realizando merge" ${Originbranch} y ${destinybranch}
    
    sh """
        git merge ${Originbranch}
        git push origin ${destinybranch}
        """
}

def checkout (String branch) {
    sh "git reset --hard HEAD; git checkout ${branch}; git pull origin ${branch}"
}
