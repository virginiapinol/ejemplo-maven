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
        stage('SonarQube analysis') {
        withSonarQubeEnv(credentialsId: 'f225455e-ea59-40fa-8af7-08176e86507a', installationName: 'My SonarQube Server') { // You can override the credential to be used
          sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
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

withSonarQubeEnv('sonarVirginia', envOnly: true) {
  // This expands the evironment variables SONAR_CONFIG_NAME, SONAR_HOST_URL, SONAR_AUTH_TOKEN that can be used by any script.
  println ${env.SONAR_HOST_URL} 
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
