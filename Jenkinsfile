pipeline {
    agent any

    tools {
        jdk 'java21'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/madhumallela99/FullStack-Blogging-App.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Test & JaCoCo') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {

                    sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=blogging-app \
                    -Dsonar.projectName=blogging-app \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        

        stage('Trivy FS Scan') {
            steps {
                sh '''
                trivy fs \
                --format table \
                . > trivy-fs-report.txt
                '''
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Publish Artifact To Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'maven-settings',
                    jdk: 'java21',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh 'mvn deploy -DskipTests'
                }
            }
        }
        stage('Docker build') {
            steps { 
                script{
        withDockerRegistry(credentialsId: 'docker-creds') {
            sh " docker build -t madhumallela/bloggingapp:latest ."

}

    }
            }
        }
       stage('Trivy Image Scan') {
    steps {
        sh '''
        export TMPDIR=/var/tmp
        trivy image --format table -o image.html madhumallela/bloggingapp:latest
        '''
    }
}
        stage('Docker push') {
            steps { 
                script{
        withDockerRegistry(credentialsId: 'docker-creds') {
            sh " docker push madhumallela/bloggingapp:latest "

}

    }
            }
        }
    }

    post {

        always {

            archiveArtifacts(
                artifacts: 'trivy-fs-report.txt',
                allowEmptyArchive: true
            )

            archiveArtifacts(
                artifacts: 'target/site/jacoco/*',
                allowEmptyArchive: true
            )
        }

        success {
            echo 'Pipeline executed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}
