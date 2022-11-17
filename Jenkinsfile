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
        stage('Análisis Sonarqube') {
            environment {
                scannerHome = tool 'sonarVirginia'
            }
            steps {
                 withSonarQubeEnv('sonarVirginia') {
                    sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=ejemplo_maven -Dsonar.host.url=http://sonarqube_vp:9000 -Dsonar.login=sqp_4837d908102b5c23f61e7fe24a87a922e26858ec'
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
