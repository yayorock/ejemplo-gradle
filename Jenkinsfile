import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

pipeline {
    agent any
    environment {
        NEXUS_USER         = credentials('nexus_user')
        NEXUS_PASSWORD     = credentials('nexus_pass')
    }
    stages {
        stage("Paso 1: Build && Test"){
            steps{
                script {
                    sh "echo 'Build && Test!'"
                    sh "gradle clean build"
                }
            }
        }

        stage("Paso 2: Sonar - Análisis Estático"){
            steps{
                script {
                    sh "echo 'Análisis Estático!'"
                    withSonarQubeEnv('sonarqube') {
                        sh "echo 'Calling sonar by ID!'"
                        // Run Maven on a Unix agent to execute Sonar.
                        sh './gradlew sonarqube -Dsonar.projectKey=ejemplo-gradle -Dsonar.java.binaries=build'
                    }
                }
            }
        }
        stage("Paso 3: Curl Springboot Gradle sleep 20"){
            steps{
                script {
                    sh "gradle bootRun&"
                }
            }
        }
        stage("Paso 4: Subir Nexus"){
            steps{
                script {
                    nexusPublisher nexusInstanceId: 'nexus3',
                    nexusRepositoryId: 'devops-usach-nexus',
                    packages: [
                        [$class: 'MavenPackage',
                            mavenAssetList: [
                                [classifier: '',
                                extension: '.jar',
                                filePath: 'build/libs/DevOpsUsach2020-0.0.1.jar'
                            ]
                        ],
                            mavenCoordinate: [
                                artifactId: 'DevOpsUsach2020',
                                groupId: 'com.devopsusach2020',
                                packaging: 'jar',
                                version: '0.0.1'
                            ]
                        ]
                    ]
                }
            }
        }
        stage("Paso 5: Descargar Nexus"){
            steps{
                script {
                    sh ' curl -X GET -u $NEXUS_USER:$NEXUS_PASSWORD "http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }
            }
        }
        stage("Paso 6: Levantar Artefacto Jar"){
            steps{
                script {
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
        stage("Paso 7: Testear Artefacto - Dormir(Esperar 20sg) "){
            steps{
                script {
                    sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
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
