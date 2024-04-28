@Library('sharedlibrary')_

pipeline {
    environment {
        gitRepoURL = "${env.GIT_URL}"
        gitBranchName = "${env.BRANCH_NAME}"
        repoName = sh(script: "basename -s .git ${GIT_URL}", returnStdout: true).trim()
        dockerImage = "rubinafayeen58/two-tier:latest" // Updated Docker Hub repository
        branchName = sh(script: 'echo $BRANCH_NAME | sed "s#/#-#"', returnStdout: true).trim()
        gitCommit = "${GIT_COMMIT[0..6]}"
        dockerTag = "${branchName}-${gitCommit}"
        snykOrg = "42babfb3-cd5a-488b-b2da-d7af6ac6d426"
        SCANNER_HOME=tool 'sonar-scanner'
        dockerHubCred = 'docker' // Docker Hub credential ID
    }

    agent {label 'docker'}
    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/Rubeenakhan16/nov23-tetris-game.git'
            }
        }

        // Remaining stages remain unchanged

        stage('Docker Build') {
            steps {
                dockerImageBuild("${dockerImage}:${dockerTag}")
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: dockerHubCred, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', DOCKER_USERNAME, DOCKER_PASSWORD) {
                            dockerPush("${dockerImage}:${dockerTag}") // Pushing to Docker Hub
                        }
                    }
                }
            }
        }

        stage('Kubernetes Deploy - DEV') {
            when {
                branch 'development'
            }
            steps {
                kubernetesEKSHelmDeployEnv("$dockerImage", "$dockerTag", "$repoName", "awsCred", "ap-south-1", "eks-cluster", "dev")
            }
        }

        stage('Kubernetes Deploy - UAT') {
            when {
                branch 'master_staging'
            }
            steps {
                kubernetesEKSHelmDeployEnv("$dockerImage", "$dockerTag", "$repoName", "awsCred", "ap-south-1", "eks-cluster", "uat")
            }
        }

        stage('Kubernetes Deploy - PROD') {
            when {
                branch 'master'
            }
            steps {
                kubernetesEKSHelmDeployEnv("$dockerImage", "$dockerTag", "$repoName", "awsCred", "ap-south-1", "eks-cluster", "prod")
            }
        }
    }
}

