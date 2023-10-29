pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages{
        stage('git checkout'){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/mukeshr-29/Projects-13-Tetris-game-V1.git'
            }
        }
        stage('sonar analysis'){
    steps{
        withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'sonarqube'){
            sh '''
                $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=tetris-v1 \
                -Dsonar.projectKey=tetris-v1
            '''
                }
            }
        }
        stage('quality gate'){
            steps{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
            }
        }
        stage('install npm'){
            steps{
                sh 'npm install'
            }
        }
        stage('trivy scan'){
            steps{
                sh 'trivy fs .'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('docker build'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                    sh '''
                        docker build -t tetris-game-v1 .
                        docker tag tetris-game-v1 mukeshr29/tetris-game-v1:latest
                        docker push mukeshr29/tetris-game-v1:latest
                    '''
                    }
                }
            }
        }
        stage('trivy img scan'){
            steps{
                sh 'trivy image mukeshr29/tetris-game-v1:latest'
            }
        }
    }
}