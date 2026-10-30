pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Zaeem-Hassan/laravel_demo.git'
            }
        }

        stage('Copy Environment File') {
            steps {
                sh 'cp .env.example .env'
                sh 'sed -i "s/DB_CONNECTION=sqlite/DB_CONNECTION=mysql/" .env'
                sh 'sed -i "s/# DB_HOST=127.0.0.1/DB_HOST=db/" .env'
                sh 'sed -i "s/# DB_PORT=3306/DB_PORT=3306/" .env'
                sh 'sed -i "s/# DB_DATABASE=laravel/DB_DATABASE=laravel/" .env'
                sh 'sed -i "s/# DB_USERNAME=root/DB_USERNAME=laravel_user/" .env'
                sh 'sed -i "s/# DB_PASSWORD=/DB_PASSWORD=secret/" .env'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Start Containers') {
            steps {
                sh 'docker compose up -d'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'docker exec laravel-app composer install'
            }
        }

        stage('Generate App Key') {
            steps {
                sh 'docker exec laravel-app php artisan key:generate'
            }
        }

        stage('Setup Application') {
            steps {
                sh 'docker exec laravel-app chmod -R 775 storage bootstrap/cache'
                sh 'docker exec laravel-app php artisan config:cache'
                sh 'docker exec laravel-app php artisan route:cache'
                sh 'docker exec laravel-app php artisan view:cache'
            }
        }

        stage('Run Migrations') {
            steps {
                sh 'sleep 15'
                sh 'docker exec laravel-app php artisan migrate --force'
            }
        }

        stage('Verify Application') {
            steps {
                sh 'curl -f http://localhost:8000 || exit 1'
                echo 'Application is running and accessible!'
            }
        }
    }

    post {
        failure {
            sh 'docker compose down'
            echo 'Pipeline failed. Containers stopped.'
        }
    }
}
