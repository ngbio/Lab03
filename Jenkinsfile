pipeline {
    // agent any: chạy trên agent mặc định (Jenkins có Docker socket thì stage con có thể dùng image Node)
    agent any

    triggers {
        pollSCM('H/3 * * * * *')
    }

    options {
        // Giữ log build gom, tránh treo pipeline
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                // Khi job Lấy Jenkinsfile từ SCM, checkout thường đã có; giữ bước này cho rõ ràng/ một số cấu hình
                checkout scm
            }
        }

        // Dùng container Node để không cần cài Node trên máy Jenkins (cần plugin Docker Pipeline + quyền dùng docker.sock)
        stage('Install & Lint & Build') {
            agent{
                docker {
                    image 'node:22-alpine'
                    // Cùng workspace với bước checkout trên node Jenkins
                    reuseNode true
                }
            }
            steps {
                sh 'node -v && npm -v'
                sh 'npm ci'
                sh 'npm run lint'
                sh 'npm run build'
            }
        }
    }

    post {
        success {
            // Lưu thư mục build để tải về / xen trên giao diện Jenkins
            archiveArtifacts artifacts: 'dist/**/*', fingerprint: true, allowEmptyArchive: false
        }
        failure {
            echo 'Pipeline thất bại - xem log từng stage (Install & Lint & Build)'
        }
    }
}