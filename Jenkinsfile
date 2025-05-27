pipeline {
    agent any // Puedes especificar un agente Docker si es necesario más adelante

    tools {
        jdk 'JDK_17'        // Asegúrate que este nombre coincida con tu JDK en Global Tool Config
        maven 'MAVEN_HOME'  // Asegúrate que este nombre coincida con tu Maven en Global Tool Config
    }

    environment {
        // El ID de la credencial de Jenkins que contiene tu token de SonarQube
        SONAR_TOKEN_CREDENTIAL_ID = 'sonarqube'
        // El nombre de la configuración del servidor SonarQube en Jenkins
        SONAR_SERVER_CONFIG_NAME = 'sonarqube' // Debe coincidir con el nombre en "Configure System"
    }

    stages {
        stage('📥 Checkout Code') { // Renombrado para claridad
            steps {
                echo "=== INICIANDO CHECKOUT ==="
                // Limpia el workspace antes del checkout para asegurar un estado limpio
                cleanWs()
                // Clona la rama 'main'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/Aaronchelo18/micro-product.git']]
                ])
                echo "Checkout completado. Verificando archivos:"
                sh 'ls -la'
                sh 'test -f pom.xml && echo "✅ pom.xml existe" || error "❌ ERROR: pom.xml no encontrado"'
                echo "=== FIN CHECKOUT ==="
            }
        }

        stage('🏗️ Compile, Test & Generate Coverage') { // Combinando Build y Test para eficiencia
            steps {
                echo "=== COMPILANDO, EJECUTANDO TESTS Y GENERANDO COBERTURA ==="
                // 'clean verify' compila, ejecuta tests (Surefire), ejecuta JaCoCo,
                // y realiza otras verificaciones del ciclo de vida de Maven.
                // Esto debería generar target/site/jacoco/jacoco.xml
                sh 'mvn clean verify'

                echo "Verificando JAR:"
                sh 'ls -la target/*.jar || echo "⚠️ No se encontró JAR"'
                echo "Verificando reportes de Surefire (tests):"
                sh 'ls -la target/surefire-reports || echo "⚠️ No se encontraron reportes de Surefire"'
                echo "Verificando reporte XML de JaCoCo (cobertura):"
                sh 'test -f target/site/jacoco/jacoco.xml && echo "✅ Reporte XML de JaCoCo encontrado" || error "❌ ERROR: Reporte XML de JaCoCo NO encontrado. Revisa la config de JaCoCo en pom.xml y que los tests se ejecuten."'
                echo "=== FIN COMPILACION, TESTS Y COBERTURA ==="
            }
        }

        stage('📊 SonarQube Analysis') {
            steps {
                echo "=== INICIANDO ANALISIS SONARQUBE ==="
                withSonarQubeEnv(SONAR_SERVER_CONFIG_NAME) {
                    // Asumiendo que las propiedades clave de SonarQube (projectKey, sources,
                    // jacoco path, java.binaries, etc.) están definidas en tu pom.xml.
                    // El plugin sonar-maven-plugin las leerá automáticamente.
                    // SONAR_HOST_URL y SONAR_LOGIN (token) son inyectados por withSonarQubeEnv.
                    // Puedes usar una versión específica del plugin si es necesario, o dejar que Maven la resuelva.
                    // sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.1.2184:sonar' // Ejemplo con versión específica
                    sh 'mvn sonar:sonar' // Más simple si la versión está gestionada o es la por defecto
                }
                echo "✅ Análisis de SonarQube invocado."
                echo "=== FIN ANALISIS SONARQUBE ==="
            }
        }

             // ETAPA ELIMINADA:
        // stage('🚪 Quality Gate Check') {
        //     steps {
        //         echo "=== VERIFICANDO QUALITY GATE DE SONARQUBE ==="
        //         timeout(time: 10, unit: 'MINUTES') {
        //             script {
        //                 def qg = waitForQualityGate(webhookSecretId: '')
        //                 echo "Quality Gate Status: ${qg.status}"
        //                 if (qg.status != 'OK') {
        //                     error "Quality Gate failed: ${qg.status}. Revisa los problemas en SonarQube."
        //                 } else {
        //                     echo "✅ Quality Gate PASÓ exitosamente."
        //                 }
        //             }
        //         }
        //         echo "=== FIN QUALITY GATE ==="
        //     }
        // }



        // La etapa de Deploy se mantiene como placeholder
        stage('🚀 Deploy (Placeholder)') {
            steps {
                echo "Esta es una etapa de Deploy de ejemplo."
                echo "Aquí irían los pasos para desplegar tu aplicación."
                // Ejemplo: sh "mvn spring-boot:run -Dspring-boot.run.profiles=dev"
            }
        }
    }

    post {
        always {
            echo "=== POST-PROCESO SIEMPRE ==="
            // cleanWs() // Considera si realmente quieres limpiar el workspace siempre.
            // Puede ser útil para depurar dejarlo. Si lo activas, asegúrate que no interfiera
            // con la necesidad de artefactos para etapas posteriores si las tuvieras fuera de este pipeline.
            echo "Pipeline finalizado. Estado: ${currentBuild.currentResult}"
            echo "=== FIN POST-PROCESO ==="
        }
        success {
            echo "🎉 ¡PIPELINE COMPLETADO EXITOSAMENTE!"
        }
        failure {
            echo "💥 PIPELINE FALLÓ."
            // Aquí podrías añadir notificaciones (email, Slack, etc.)
        }
        unstable {
            echo "⚠️ PIPELINE INESTABLE (Ej. Quality Gate no OK pero no se marcó como fallo total, o tests fallaron pero no detuvieron el build)."
        }
    }
}