pipeline {
  agent any
  tools {
    maven 'MAVEN_3_9_11'
    jdk 'JDK_24'
  }
	environment {
        // Nombre de la imagen que vamos a crear para nuestra aplicación
        IMAGE_NAME = "learning-center-small"
        TAG        = "${env.BUILD_NUMBER}" // Usa el número de ejecución de Jenkins como versión
    }

  stages {
    stage ('Compile Project') {
      steps {
        withMaven(maven : 'MAVEN_3_9_11') {
            sh 'mvn clean compile'
        }
      }
    }

    stage('Validate Checkstyle') {
      steps {
        withMaven(maven: 'MAVEN_3_9_11') {
          sh 'mvn checkstyle:check'
        }
      }
    }

    stage('Validate Unit Tests') {
      steps {
        withMaven(maven: 'MAVEN_3_9_11') {
          sh 'mvn test'
        }
      }
    }

    stage('Validate Test Coverage') {
      steps {
        withMaven(maven: 'MAVEN_3_9_11') {
          sh 'mvn clean verify jacoco:report'
          sh 'mvn jacoco:check'
        }
      }
    }

	 stage ('SonarQube Analysis') {
        steps {
			// 1. Enviar el código a analizar a SonarQube
            withSonarQubeEnv('MiSonarServer') {
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=learning-center'
            }
			// 2. Pausar el pipeline y esperar la respuesta del Webhook de SonarQube
	        script {
	            timeout(time: 10, unit: 'MINUTES') { // Evita que se quede bloqueado permanentemente si cae la red
	                // Este paso intercepta la notificación enviada al puerto 9089
	                def qg = waitForQualityGate()
	                
	                // 3. Evaluar el estado del Quality Gate
	                if (qg.status != 'OK') {
	                    error "El pipeline se ha detenido porque el código no superó el Quality Gate de SonarQube. Estado: ${qg.status}"
	                }
	            }
	        }

        }
     }

	  stage('Construir Imagen Docker') {
            steps {
                script {
                    echo "Iniciando la construcción de la imagen de Docker: ${IMAGE_NAME}:${TAG}"
                    
                    // Ejecuta el comando de Docker utilizando el socket compartido del Host
                    // Supone que tienes un archivo 'Dockerfile' en la raíz de tu proyecto Spring Boot
                    //sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    //sh "docker build -t ${IMAGE_NAME}:latest ."

					echo "Construyendo imagen híbrida/compatible con servidores de producción (AMD64)..."
					// Usamos 'buildx' para asegurar que la imagen de salida sea estrictamente para plataformas de 64 bits estándar
					sh "docker buildx build --platform linux/amd64 -t ${IMAGE_NAME}:${TAG} --load ."
					sh "docker buildx build --platform linux/amd64 -t ${IMAGE_NAME}:latest --load ."
                    
                    echo "Imagen construida exitosamente."
                }
            }
        }
	  

    /*stage ('package Project') {
        steps {
            withMaven(maven : 'MAVEN_3_9_11') {
                sh 'mvn package'
            }
        }
    }*/


    }
}
