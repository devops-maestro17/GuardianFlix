
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Build & Tag Docker Image"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=45216d1407a8279296f7744f9d94110b -t netflix ."
                       sh "docker tag netflix .containerizeOps/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image containerizeOps/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Push to Dockerhub'){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker push containerizeOps/netflix:latest"
                    }
                }
            }
        }
        // stage("Update Image Tag in Manifests"){
        //     environment {
        //         GIT_REPO_NAME = "Board-Game-App"
        //         GIT_USER_NAME = "devops-maestro17"
        //     }
        //     steps {
        //         withCredentials([string(credentialsId: 'github-cred', variable: 'GITHUB_TOKEN')]) {
        //             sh '''
        //                 git config user.email "rajdeep_deogharia@outlook.com"
        //                 git config user.name "devops-maestro17"
        //                 BUILD_NUMBER=${BUILD_NUMBER}
        //                 sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment-service.yaml
        //                 git add deployment-service.yaml
        //                 git commit -m "Update deployment image to version ${BUILD_NUMBER}"
        //                 git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
        //             '''
        //         }
        //     }
        // }
    }
}

