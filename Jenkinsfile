
pipeline {
    agent any

    environment {
        KUBECTL = 'D:\\DockerApp\\resources\\bin\\kubectl.exe'
        KUBECONFIG = 'C:\\Users\\POOVARASAN\\.kube\\config'
    }

    stages {
        stage('Deploy Application') {
            steps {
                bat '"%KUBECTL%" apply --validate=false -f deployment.yaml'
            }
        }

        stage('Simulate Failure') {
            steps {
                bat '"%KUBECTL%" set image deployment/troubleshoot-app nginx=nginx:bad-image'
            }
        }

        stage('Detect and Troubleshoot') {
            steps {
                script {
                    def result = bat(
                        script: '"%KUBECTL%" rollout status deployment/troubleshoot-app --timeout=30s',
                        returnStatus: true
                    )

                    if (result != 0) {
                        echo 'Deployment failure detected. Collecting diagnostics...'
                        bat '"%KUBECTL%" describe deployment troubleshoot-app'
                        bat '"%KUBECTL%" get pods'
                        bat '"%KUBECTL%" get events --sort-by=.metadata.creationTimestamp'
                        echo 'Recovering deployment with a valid image...'
                        bat '"%KUBECTL%" set image deployment/troubleshoot-app nginx=nginx:stable'
                        bat '"%KUBECTL%" rollout status deployment/troubleshoot-app --timeout=120s'
                    }
                }
            }
        }

        stage('Verify Recovery') {
            steps {
                bat '"%KUBECTL%" get deployments'
                bat '"%KUBECTL%" get pods'
            }
        }
    }
}
