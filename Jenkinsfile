pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }
    
    // Define multiple configurations for different environments
    parameters {
        choice(
            name: 'DOCKER_ENV',
            choices: ['all', 'alpine-linux', 'slim', 'latest'],
            description: 'Select Docker environment(s) to test'
        )
    }
    
    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'multi-config-docker'
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '========== Stage: Checkout =========='
                checkout scm
                sh 'echo "Repository checked out successfully"'
                sh 'ls -la'
            }
        }
        
        stage('Prepare') {
            steps {
                echo '========== Stage: Prepare =========='
                sh '''
                    echo "Java Version:"
                    java -version
                    echo "Docker Version:"
                    docker --version
                    echo "Build timestamp: $(date)"
                '''
            }
        }
        
        stage('Compile Application') {
            steps {
                echo '========== Stage: Compile Application =========='
                sh '''
                    echo "Compiling App.java from alpine-linux directory..."
                    cd alpine-linux
                    javac App.java
                    echo "✓ Compilation successful"
                    cd ..
                '''
            }
        }
        
        stage('Unit Test') {
            steps {
                echo '========== Stage: Unit Test =========='
                sh '''
                    echo "Running application test..."
                    cd alpine-linux
                    java App
                    echo "✓ Application test passed"
                    cd ..
                '''
            }
        }
        
        stage('Build - Alpine Linux') {
            when {
                expression { params.DOCKER_ENV == 'all' || params.DOCKER_ENV == 'alpine-linux' }
            }
            steps {
                echo '========== Stage: Build - Alpine Linux =========='
                sh '''
                    echo "Building Alpine Linux Docker image..."
                    docker build -t ${IMAGE_NAME}:alpine -f alpine-linux/Dockerfile alpine-linux/
                    echo "✓ Alpine image built successfully"
                    docker images | grep ${IMAGE_NAME}
                '''
            }
        }
        
        stage('Build - Slim') {
            when {
                expression { params.DOCKER_ENV == 'all' || params.DOCKER_ENV == 'slim' }
            }
            steps {
                echo '========== Stage: Build - Slim =========='
                sh '''
                    echo "Building Slim Docker image..."
                    docker build -t ${IMAGE_NAME}:slim -f slim/Dockerfile slim/
                    echo "✓ Slim image built successfully"
                    docker images | grep ${IMAGE_NAME}
                '''
            }
        }
        
        stage('Build - Latest') {
            when {
                expression { params.DOCKER_ENV == 'all' || params.DOCKER_ENV == 'latest' }
            }
            steps {
                echo '========== Stage: Build - Latest =========='
                sh '''
                    echo "Building Latest Docker image..."
                    docker build -t ${IMAGE_NAME}:latest -f latest/Dockerfile latest/
                    echo "✓ Latest image built successfully"
                    docker images | grep ${IMAGE_NAME}
                '''
            }
        }
        
        stage('Test - Alpine Linux') {
            when {
                expression { params.DOCKER_ENV == 'all' || params.DOCKER_ENV == 'alpine-linux' }
            }
            steps {
                echo '========== Stage: Test - Alpine Linux =========='
                sh '''
                    echo "Testing Alpine Linux image..."
                    docker run --rm ${IMAGE_NAME}:alpine
                    echo "✓ Alpine Linux image test passed"
                '''
            }
        }
        
        stage('Test - Slim') {
            when {
                expression { params.DOCKER_ENV == 'all' || params.DOCKER_ENV == 'slim' }
            }
            steps {
                echo '========== Stage: Test - Slim =========='
                sh '''
                    echo "Testing Slim image..."
                    docker run --rm ${IMAGE_NAME}:slim
                    echo "✓ Slim image test passed"
                '''
            }
        }
        
        stage('Test - Latest') {
            when {
                expression { params.DOCKER_ENV == 'all' || params.DOCKER_ENV == 'latest' }
            }
            steps {
                echo '========== Stage: Test - Latest =========='
                sh '''
                    echo "Testing Latest image..."
                    docker run --rm ${IMAGE_NAME}:latest
                    echo "✓ Latest image test passed"
                '''
            }
        }
        
        stage('Image Inspection') {
            steps {
                echo '========== Stage: Image Inspection =========='
                sh '''
                    echo "Inspecting built images..."
                    echo "===== Alpine Image ====="
                    docker inspect ${IMAGE_NAME}:alpine 2>/dev/null || echo "Alpine image not built in this run"
                    echo ""
                    echo "===== Slim Image ====="
                    docker inspect ${IMAGE_NAME}:slim 2>/dev/null || echo "Slim image not built in this run"
                    echo ""
                    echo "===== Latest Image ====="
                    docker inspect ${IMAGE_NAME}:latest 2>/dev/null || echo "Latest image not built in this run"
                '''
            }
        }
        
        stage('Generate Report') {
            steps {
                echo '========== Stage: Generate Report =========='
                sh '''
                    echo "Build Summary Report" > build-report.txt
                    echo "===================" >> build-report.txt
                    echo "Build Number: ${BUILD_NUMBER}" >> build-report.txt
                    echo "Build URL: ${BUILD_URL}" >> build-report.txt
                    echo "Build Timestamp: $(date)" >> build-report.txt
                    echo "Selected Environment(s): ${DOCKER_ENV}" >> build-report.txt
                    echo "" >> build-report.txt
                    echo "Docker Images Built:" >> build-report.txt
                    docker images | grep ${IMAGE_NAME} >> build-report.txt || echo "No images built" >> build-report.txt
                    echo "" >> build-report.txt
                    echo "Status: SUCCESS" >> build-report.txt
                    cat build-report.txt
                '''
            }
        }
    }
    
    post {
        success {
            echo '✓ ========== PIPELINE SUCCESSFUL =========='
            sh '''
                echo "All tests passed successfully!"
                docker images | grep ${IMAGE_NAME}
            '''
            archiveArtifacts artifacts: 'build-report.txt', allowEmptyArchive: true
        }
        failure {
            echo '✗ ========== PIPELINE FAILED =========='
            sh '''
                echo "Pipeline failed. Check logs above for details."
                docker images | grep ${IMAGE_NAME} || echo "No images found"
            '''
        }
        unstable {
            echo '⚠ ========== PIPELINE UNSTABLE =========='
        }
        always {
            echo '========== Cleanup =========='
            sh '''
                echo "Cleaning up Docker resources..."
                docker system prune -f --volumes || true
            '''
        }
    }
}
