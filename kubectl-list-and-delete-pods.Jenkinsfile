@Library("com.vvorg.jenkins.pipeline.library@master") _

def k8sNamespace = 'gp-fulfillment-hub'
def labelSelector = 'kubernetes_pod_operator=True'

pipeline {
    agent {
        label 'docker-terraform-agent'
    }
    environment {
        AZURE_NON_PROD_SERVICE_PRINCIPAL = "AZURE_SERVICE_PRINCIPAL_NP"
        AZURE_PROD_SERVICE_PRINCIPAL = "AZURE_SERVICE_PRINCIPAL_PROD"
    }
    stages {

        stage('Clean up Dev') {
            steps {
                script {
                    glAzureLogin([credentialsId: env.AZURE_NON_PROD_SERVICE_PRINCIPAL]) {
                        command """
                            az aks get-credentials -g gpfhub-dev-rg -n gpfhub-dev-akscluster
                            kubelogin convert-kubeconfig -l azurecli
                            runningPods=\$(kubectl get pods -n ${k8sNamespace} -l ${labelSelector} --field-selector=status.phase=Running -o jsonpath='{.items[*].metadata.name}' 2>/dev/null)
                            
                            for pod in \${runningPods}; do
                                containerLog=\$(kubectl logs -2f \${pod} -n ${k8sNamespace} 2>/dev/null)
                                echo \$containerLog
                            done
                        """
                    }
                }

            }
        }

    }
}
