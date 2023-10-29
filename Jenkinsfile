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
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/mukeshr-29/Project-13-Tetris-game-V2.git'
            }
        }
        stage('sonar analysis'){
    steps{
        withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'sonarqube'){
            sh '''
                $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=tetris-v2 \
                -Dsonar.projectKey=tetris-v2
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
                        docker build -t tetris-game-v2 .
                        docker tag tetris-game-v1 mukeshr29/tetris-game-v2:latest
                        docker push mukeshr29/tetris-game-v2:latest
                    '''
                    }
                }
            }
        }
        stage('trivy img scan'){
            steps{
                sh 'trivy image mukeshr29/tetris-game-v2:latest'
            }
        }
        stage('trigger manifest'){
            steps{
                build job: 'tetris-manifest', wait:true
            }
        }
        // stage('git checkout manifest'){
        //     steps{
        //         git branch: 'main', credentialsId: 'github', url: 'https://github.com/mukeshr-29/Project-13-Tetris-game-manifest.git'
        //     }
        // }
        // stage('update image'){
        //     steps{
        //         script{
        //             withCredentials([string(credentialsId: 'github1', variable: 'GITHUB_TOKEN')]){
        //                 NEW_IMAGE_NAME = "mukeshr29/tetris-game-v1:latest"
        //                 sh "sed -i 's|image: .*|image: $NEW_IMAGE_NAME|' deployment.yml"
        //                 sh 'git add deployment.yml'
        //                 sh "git commit -m 'Update deployment image to $NEW_IMAGE_NAME'"
        //                 sh "git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main"
        //             }
        //         }
        //     }
        // }
    }
}