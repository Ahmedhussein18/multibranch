pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ahmedhussein18/ivolve-image"
        DOCKER_TAG = "latest"  // Or use a dynamic tag like BUILD_ID or a timestamp
        IMAGE_FULL_NAME = "${DOCKER_IMAGE}:${DOCKER_TAG}"
        KUBE_DEPLOY_FILE = "deployment.yaml"
        GIT_CREDENTIALS_ID = 'git-repo-credentials'
    }
    stages {
        // Previous stages remain unchanged

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-access-token', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        sh "docker push ${IMAGE_FULL_NAME}"
                    }
                }
            }
        }
        stage('Update K8s Deployment') {
            steps {
                script {
                    // Checkout the Git repository that Argo CD is monitoring
                    git credentialsId: GIT_CREDENTIALS_ID, url: 'https://github.com/Ahmedhussein18/multibranch.git'

                    // Update the Kubernetes deployment.yaml file to use the new Docker image
                    sh "sed -i 's|image:.*|image: ${IMAGE_FULL_NAME}|' ${KUBE_DEPLOY_FILE}"

                    // Commit and push the changes
                    sh """
                      git config user.email 'jenkins@yourdomain.com'
                      git config user.name 'Jenkins'
                      git add ${KUBE_DEPLOY_FILE}
                      git commit -m 'Update image to ${IMAGE_FULL_NAME}'
                      git push
                    """
                }
            }
        }
    }
}
