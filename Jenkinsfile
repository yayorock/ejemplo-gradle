import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

pipeline {
    agent any
    environment {
        NEXUS_USER_VAR = credentials('nexus_user')
        NEXUS_USER_PASS_VAR = credentials('nexus_pass')
    }
    stages {
        stage('compile') {
            steps {
                script {
                    sh "echo 'Compilando el Código'"
                    sh "mvn clean compile -e"
                }
            }
        }
        stage('test') {
            steps {
                script {
                    sh "echo 'Testenado el codigo'"
                    sh "mvn clean test -e"
                }
            }
        }
        stage('jar') {
            steps {
                script {
                    sh "echo 'Generando el jar'"
                    sh "mvn clean package -e"
                }
            }
            post {
                success {
                    archiveArtifacts artifacts:'build/*.jar'
                }
            }
        }
        stage('sonar') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "echo 'Llamando a sonarqube'"
                    // Run Maven on a Unix agent to execute Sonar.
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }
        stage('uploadNexus') {
            steps {
                script {
                    sh "echo 'Subiendo a nexus...'"
                    nexusPublisher nexusInstanceId: 'nexus', 
                    nexusRepositoryId: 'devops-usach-nexus', 
                    packages: [
                        [$class: 'MavenPackage', 
                            mavenAssetList: [
                                [classifier: '', 
                                extension: '.jar', 
                                filePath: '/var/jenkins_home/workspace/job-taller-10/build/DevOpsUsach2020-0.0.1.jar']
                            ], 
                            mavenCoordinate: [
                                artifactId: 'DevOpsUsach2020', 
                                groupId: 'com.devopsusach2020', 
                                packaging: 'jar', 
                                version: '1.0.0']
                        ]
                    ]
                }
            }
        }
        stage('Bajar de nexus'){
            steps {
                script {
                    sh ' echo "Bajando jar de nexus.."'
                    sh ' curl -X GET -u $NEXUS_USER_VAR:$NEXUS_USER_PASS_VAR "http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }
            }
        }
        stage('Test del Build'){
            steps {
                script {
                    sh ' echo "Iniciando..."'
                    sh ' nohup java -jar DevOpsUsach2020-0.0.1.jar & > /dev/null'
                    sh " sleep 30 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }
    }
    post {
        always {
            sh "echo 'fase always executed post'"
        }
        success {
            sh "echo 'fase success'"
        }
        failure {
            sh "echo 'fase failure'"
        }
    }
}
