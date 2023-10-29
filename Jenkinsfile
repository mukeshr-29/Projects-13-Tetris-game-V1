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
                withSonarQubeEnv(credentialsId: 'sonarqube'){
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
    }
}