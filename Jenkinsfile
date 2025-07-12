/*
 * Jenkins Declarative Pipeline to deploy PostgreSQL to Kubernetes.
 *
 * This pipeline runs on a dynamically provisioned Kubernetes pod agent
 * that has `kubectl` installed. This ensures the necessary tools are
 * available to interact with the cluster.
 */
pipeline {
    // Define a Kubernetes pod as the agent for this pipeline.
    agent {
        label 'kubectl'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // This step checks out your code from the Git repository
                // where your Jenkinsfile and YAML files are stored.
                checkout scm
            }
        }

        stage('Deploy PostgreSQL to Minikube') {
            steps {
                script {
                    echo "Applying PostgreSQL manifests to the cluster..."

                    // The order of application is important to ensure dependencies are met.
                    // 1. ConfigMap for configuration data.
                    // 2. PersistentVolumeClaim for storage.
                    // 3. Deployment to create the pod.
                    // 4. Service to expose the pod.
                    try {
                        sh 'kubectl apply -f postgres-configmap.yaml'
                        sh 'kubectl apply -f postgres-pvc.yaml'
                        sh 'kubectl apply -f postgres-deployment.yaml'
                        sh 'kubectl apply -f postgres-service.yaml'

                        echo "All manifests applied successfully."

                        // This is an optional but recommended step.
                        // It waits for the deployment to complete successfully
                        // before marking the stage as complete.
                        echo "Verifying deployment rollout status..."
                        sh 'kubectl rollout status deployment/postgres-deployment --timeout=120s'

                    } catch (e) {
                        // If any command fails, fail the pipeline stage
                        currentBuild.result = 'FAILURE'
                        error "Failed to deploy PostgreSQL: ${e.message}"
                    }
                }
            }
        }
    }

    post {
        always {
            // This block runs regardless of the pipeline's outcome.
            echo 'Pipeline finished.'
        }
        success {
            echo 'PostgreSQL was deployed successfully!'
        }
        failure {
            echo 'PostgreSQL deployment failed. Please check the logs.'
        }
    }
}

