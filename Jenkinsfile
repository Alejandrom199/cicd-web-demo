pipeline {
    agent any

    environment {
        APP_NAME = "cicd-web-demo" // [cite: 239]
        STAGING_PORT = "8081"      // [cite: 240]
        PROD_PORT = "8082"         // [cite: 241]
    }

    options {
        timestamps()               // [cite: 245]
        disableConcurrentBuilds()  // [cite: 246]
    }

    stages {
        stage('Checkout') {        // [cite: 250]
            steps {
                echo "Descargando el código..."
                // Jenkins ya hace checkout si el job está configurado con SCM 
            }
        }

        stage('Lint / Validación') { // [cite: 258]
            steps {
                echo "Validando estructura mínima..."
                sh 'test -f Dockerfile'         // [cite: 261]
                sh 'test -f docker-compose.yml' // [cite: 262]
                sh 'test -f app/index.html'     // [cite: 263]
                sh 'test -x scripts/test.sh'    // [cite: 264]
                echo "Validación OK"
            }
        }

        stage('Test') { // [cite: 268]
            steps {
                echo "Ejecutando pruebas..."
                sh './scripts/test.sh' // [cite: 271]
            }
        }

        stage('Build Imagen (staging)') { // [cite: 274]
            steps {
                echo "Construyendo imagen para staging..."
                sh "docker build -t ${APP_NAME}:staging ." // [cite: 277]
            }
        }

        stage('Deploy a Staging') { // [cite: 280]
            steps {
                echo "Desplegando en STAGING (puerto ${STAGING_PORT})..."
                // Levanta/actualiza solo el servicio staging 
                sh 'docker compose up -d web-staging' // [cite: 284]
                echo "Staging actualizado."
            }
        }

        stage('Aprobación para Producción') { // [cite: 288]
            steps {
                // El pipeline se detiene aquí para aprobación manual [cite: 293, 348]
                input message: '¿Aprobar despliegue a PRODUCCIÓN?', ok: 'Sí, desplegar'
            }
        }

        stage('Promover Imagen a Producción') { // [cite: 296]
            steps {
                echo "Promoviendo imagen a producción..."
                // Re-etiquetar staging como production [cite: 299, 300]
                sh "docker tag ${APP_NAME}:staging ${APP_NAME}:production"
            }
        }

        stage('Deploy a Producción') { // [cite: 303]
            steps {
                echo "Desplegando en PRODUCCIÓN (puerto ${PROD_PORT})..."
                sh 'docker compose up -d web-production' // [cite: 306]
                echo "Producción actualizada."
            }
        }
    }

    post {
        success {
            echo "CI/CD completado con éxito." // [cite: 313]
        }
        failure {
            echo "CI/CD falló. Revisar logs del build." // [cite: 316]
        }
        always {
            // Muestra el estado de los contenedores al finalizar [cite: 319]
            sh 'docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}" || true'
        }
    }
}