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
    parameters {
        choice(
            name: 'compileTool',
            choice: ['Maven', 'Gradle'],
            description : 'Seleccione herramienta de compilacion'
        )
    }
    stages {
        stage('Pipeline') {
            steps {
                script {
                    switch(params.compileTool)
                    {
                        case 'Maven':
                            def ejecucion = load 'maven.groovy'
                            ejecucion.call()
                        break;
                        case 'Gradle':
                            def ejecucion = load 'gradle.groovy'
                            ejecucion.call()
                        break;
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
    }
}
