pipeline {
    agent any

    tools {
        // Usa la instalación de Maven configurada en Jenkins como "MAVEN_HOME"
        maven "MAVEN_HOME"
    }

    stages {
        stage('Clone') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    git branch: 'master', url: 'https://github.com/Aaronchelo18/micro-product.git'
                }
            }
        }

        stage('Build') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sh "mvn -DskipTests clean package"
                }
            }
        }

        stage('Test') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sh "mvn clean install"
                }
            }
        }

        stage('Sonar') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    withSonarQubeEnv('sonarqube') {
                        sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar -Pcoverage"
                    }
                }
            }
        }

        stage('Quality gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sleep(time: 10, unit: 'SECONDS') // Espera breve para que Sonar procese
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Comando de deploy sería algo como: mvn spring-boot:run"
                // Descomenta la siguiente línea si deseas ejecutar el proyecto
                // sh "mvn spring-boot:run"
            }
        }
    }
}