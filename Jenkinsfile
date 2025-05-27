pipeline {
    agent any

    tools {
        jdk 'JDK_17'     // CONFIRMA: El nombre de tu JDK en Jenkins Global Tool Config
        maven 'MAVEN_HOME' // CONFIRMA: El nombre de tu Maven en Jenkins Global Tool Config
    }

    environment {
        SONAR_TOKEN_CREDENTIAL_ID = 'sonarqube' // ID de tu credencial de SonarQube en Jenkins
        // SONAR_HOST_URL se tomará de la configuración del servidor SonarQube en Jenkins
        SONAR_SERVER_CONFIG_NAME = 'SonarQubeServerLocal' // CONFIRMA: Nombre de tu config de SonarQube Server en Jenkins
    }

    stages {
        stage('🔍 Environment Check') {
            steps {
                echo "=== VERIFICANDO ENTORNO ==="
                sh 'java -version'
                sh 'mvn -version'
                echo "Workspace: ${WORKSPACE}"
                sh 'pwd'
                sh 'ls -la'
                echo "=== FIN VERIFICACION ENTORNO ==="
            }
        }

        stage('📥 Checkout') {
            steps {
                echo "=== INICIANDO CHECKOUT ==="
                checkout scm
                echo "Checkout completado. Verificando archivos:"
                sh 'ls -la'
                sh 'test -f pom.xml && echo "✅ pom.xml existe" || echo "❌ ERROR: pom.xml no encontrado"'
                echo "=== FIN CHECKOUT ==="
            }
        }

        stage('🏗️ Build & Unit Test (with Coverage)') {
            steps {
                echo "=== COMPILANDO, EJECUTANDO TESTS Y GENERANDO COBERTURA ==="
                // 'clean package' compila, ejecuta tests (Surefire), y JaCoCo genera el reporte de cobertura
                sh 'mvn clean package'
                echo "Verificando JAR:"
                sh 'ls -la target/*.jar || echo "⚠️ No se encontró JAR"'
                echo "Verificando reportes de Surefire (tests):"
                sh 'ls -la target/surefire-reports || echo "⚠️ No se encontraron reportes de Surefire"'
                echo "Verificando reporte XML de JaCoCo (cobertura):"
                sh 'test -f target/site/jacoco/jacoco.xml && echo "✅ Reporte XML de JaCoCo encontrado" || echo "❌ ERROR: Reporte XML de JaCoCo NO encontrado. Revisa la config de JaCoCo en pom.xml y que los tests se ejecuten."'
                echo "=== FIN BUILD & UNIT TEST ==="
            }
        }

        stage('📊 SonarQube Analysis') {
            steps {
                echo "=== INICIANDO ANALISIS SONARQUBE ==="
                // 'withSonarQubeEnv' usa la configuración del servidor SonarQube de Jenkins
                // (que incluye la URL y el token si está asociado a la credencial correcta)
                withSonarQubeEnv(SONAR_SERVER_CONFIG_NAME) {
                    // Como las propiedades principales de SonarQube (projectKey, sources, jacoco path, etc.)
                    // están en el pom.xml, el comando es simple.
                    // El plugin sonar-maven-plugin leerá esas propiedades.
                    // SONAR_HOST_URL y SONAR_LOGIN (token) son inyectados por withSonarQubeEnv.
                    sh 'mvn sonar:sonar'
                }
                echo "✅ Análisis de SonarQube invocado."
                echo "=== FIN ANALISIS SONARQUBE ==="
            }
        }

        stage('🚪 Quality Gate') {
            steps {
                echo "=== VERIFICANDO QUALITY GATE DE SONARQUBE ==="
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        // Espera a que el análisis en SonarQube (desencadenado en el paso anterior) se complete
                        // y luego verifica el estado del Quality Gate.
                        // webhookSecretId se deja vacío si no usas webhooks para la notificación del Quality Gate.
                        def qg = waitForQualityGate(webhookSecretId: '')
                        echo "Quality Gate Status: ${qg.status}"

                        if (qg.status != 'OK') {
                            echo "❌ Quality Gate FALLÓ: ${qg.status}"
                            // Puedes hacer que el build falle o sea inestable
                            // currentBuild.result = 'UNSTABLE'
                            error "Quality Gate failed: ${qg.status}. Revisa los problemas en SonarQube."
                        } else {
                            echo "✅ Quality Gate PASÓ exitosamente."
                        }
                    }
                }
                echo "=== FIN QUALITY GATE ==="
            }
        }
    }

    post {
        always {
            echo "=== POST-PROCESO SIEMPRE ==="
            echo "Limpiando workspace..."
            cleanWs()
            echo "✅ Workspace limpio."
            echo "=== FIN POST-PROCESO ==="
        }
        success {
            echo "🎉 ¡PIPELINE COMPLETADO EXITOSAMENTE!"
        }
        failure {
            echo "💥 PIPELINE FALLÓ."
        }
        unstable {
            echo "⚠️ PIPELINE INESTABLE."
        }
    }
}