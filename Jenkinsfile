import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        channel = 'C04BPL2A5E3'
        NEXUS_PASSWORD     = credentials('admin-nexus')
    }
    stages {
        /* stage("Paso 0: Download Code and checkout"){
            steps {
                script{
                    checkout(
                            [$class: 'GitSCM',
                            //Acá reemplazar por el nonbre de branch
                            branches: [[name: "feature/newman" ]],
                            //Acá reemplazar por su propio repositorio
                            userRemoteConfigs: [[url: 'https://github.com/Bfal10/ejemplo-maven.git']]])
                }
            }
        } */

        stage('Paso 1: Build && Test') {
            steps {
                script {
                    sh "echo 'Build && Test!'"
                    sh './mvnw clean package -e'
                }
            }
        }

        /* stage('Paso 2: Sonar - Análisis Estático') {
            steps {
                script {
                    sh "echo 'Análisis Estático!'"
                        withSonarQubeEnv('sonarqube') {
                            sh "echo 'Calling sonar by ID!'"
                            // Run Maven on a Unix agent to execute Sonar.
                            sh './mvnw clean verify sonar:sonar -Dsonar.projectKey=ejemplo-maven-full-stages -Dsonar.projectName=cejemplo-maven-full-stages -Dsonar.java.binaries=build'
                        }
                }
            }
        } */

        stage('Paso 3: Curl Springboot maven sleep 20') {
            steps {
                script {
                    sh 'nohup bash ./mvnw spring-boot:run  & >/dev/null'
                    sh "sleep 60 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                }
            }
        }

        stage('Paso 4: Detener Spring Boot') {
            steps {
                script {
                    sh '''
                        echo 'Process Spring Boot Java: ' $(pidof java | awk '{print $1}')
                        sleep 20
                        kill -9 $(pidof java | awk '{print $1}')
                    '''
                }
            }
        }

        /* stage('Paso 5: Subir Artefacto a Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'nexus:8081',
                    groupId: 'com.devopsusach2020',
                    version: '0.0.1',
                    repository: 'maven-usach-ceres',
                    credentialsId: 'admin-nexus',
                    artifacts: [
                        [artifactId: 'DevOpsUsach2020',
                        classifier: '',
                         file: 'build/DevOpsUsach2020-0.0.1.jar',
                        type: 'jar']
                    ]
                )
            }
        } */

        /* stage('Paso 6: Descargar Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'admin_nexus', passwordVariable: 'NXS_PASSWORD', usernameVariable: 'NXS_USERNAME')]) {
                    sh ' curl -X GET -u $NXS_USERNAME:$NXS_PASSWORD "http://nexus:8081/repository/maven-usach-ceres/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
                }
            //script{
            //     sh ' curl -X GET -u admin:$NEXUS_PASSWORD "http://nexus:8081/repository/maven-usach-ceres/com/devopsusach2020/DevOpsUsach2020/0.0.1/DevOpsUsach2020-0.0.1.jar" -O'
            //}
            }
        } */

        stage('Paso 7: Levantar Artefacto Jar en server Jenkins') {
            steps {
                script {
                    sh 'nohup java -jar DevOpsUsach2020-0.0.1.jar & >/dev/null'
                }
            }
        }
        stage('Paso 8: Testear Artefacto - Dormir(Esperar 20sg) ') {
            steps {
                script {
                    //sh "sleep 20 && curl -X GET 'http://localhost:8081/rest/mscovid/test?msg=testing'"
                    //sh 'sleep 20 && newman run ejemplo-maven.postman_collection.json'
                    sh 'echo "Testeando Artefacto"'
                }
            }
        }
        stage('Paso 9:Detener Atefacto jar en Jenkins server') {
            steps {
                sh '''
                    echo 'Process Java .jar: ' $(pidof java | awk '{print $1}')
                    sleep 20
                    kill -9 $(pidof java | awk '{print $1}')
                '''
            }
        }
    }
}
