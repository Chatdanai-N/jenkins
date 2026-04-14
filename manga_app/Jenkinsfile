pipeline {
    agent any
    
    // ตั้งค่า Parameters เพื่อให้เลือก Tag ได้
    parameters {
        string(name: 'GIT_TAG', defaultValue: 'main', description: 'ระบุชื่อ Tag หรือ Branch')
    }

    environment {
        IMAGE_NAME = "manga_app"
        IMAGE_TAG = "${params.GIT_TAG}-${env.BUILD_ID}"
        DOCKER_CONTAINER_NAME = "manga-app-container"
    }

    stages {
        stage('Checkout') {
            steps {
                // เปลี่ยน URL เป็นของโปรเจกต์คุณจริงๆ
                git url: 'https://github.com/your-user/manga_app.git', branch: params.GIT_TAG
            }
        }

        stage('Build & Test') {
            steps {
                // รัน Maven build ผ่าน wrapper ในโปรเจกต์
                sh './mvnw clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                // สร้าง Image ตาม Dockerfile ที่เตรียมไว้
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // หยุดตัวเก่าและรันตัวใหม่ใน dev-network
                    sh "docker stop ${DOCKER_CONTAINER_NAME} || true"
                    sh "docker rm ${DOCKER_CONTAINER_NAME} || true"
                    sh """
                        docker run -d \
                        --name ${DOCKER_CONTAINER_NAME} \
                        --network dev-network \
                        -p 8081:8080 \
                        ${IMAGE_NAME}:latest
                    """
                }
            }
        }
    }
}