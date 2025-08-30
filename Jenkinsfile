pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = credentials('dockerhub-username')   // Jenkins credentials ID for DockerHub username
        DOCKERHUB_PASSWORD = credentials('dockerhub-password')   // Jenkins credentials ID for DockerHub password
        GIT_TOKEN          = credentials('git-token')            // Jenkins credentials ID for GitHub token
        IMAGE_TAG          = "${env.BUILD_ID}"                   // Similar to GitHub run_id
    }

    triggers {
        pollSCM('H/5 * * * *') // Optional: poll GitHub every 5 minutes. You can replace with webhook trigger.
    }

    options {
        skipDefaultCheckout(true) // We'll checkout manually
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set up Go') {
            steps {
                sh """
                wget https://golang.org/dl/go1.22.0.linux-amd64.tar.gz
                sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz
                export PATH=$PATH:/usr/local/go/bin
                go version
                """
            }
        }

        stage('Build') {
            steps {
                sh "go build -o go-web-app"
            }
        }

        stage('Test') {
            steps {
                sh "go test ./..."
            }
        }

        stage('Code Quality - golangci-lint') {
            steps {
                sh """
                curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s v1.56.2
                ./bin/golangci-lint run ./...
                """
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    sh """
                    echo "${DOCKERHUB_PASSWORD}" | docker login -u "${DOCKERHUB_USERNAME}" --password-stdin
                    docker build -t ${DOCKERHUB_USERNAME}/go-web-app:${IMAGE_TAG} .
                    docker push ${DOCKERHUB_USERNAME}/go-web-app:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Helm Chart Tag') {
            steps {
                sh """
                git config --global user.email "nainala_sandeep@yahoo.com"
                git config --global user.name "sandeep nainala"
                sed -i 's/tag: .*/tag: "${IMAGE_TAG}"/' helm/go-web-app-chart/values.yaml
                git add helm/go-web-app-chart/values.yaml
                git commit -m "Update tag in Helm chart"
                git push https://$GIT_TOKEN@github.com/<your-repo-owner>/<your-repo-name>.git HEAD:main
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
