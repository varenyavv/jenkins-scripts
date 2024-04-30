pipeline {
    agent any
    
    stages {
        stage('Cleanup Pods') {
            steps {
                script {
                    // Define Kubernetes namespace and label selector for identifying pods to clean up
                    def namespace = 'your-namespace'
                    def labelSelector = 'kubernetes_pod_operator=True'

                    // Use kubectl to list running pods with the specified label selector in the specified namespace
                    def runningPods = sh(script: "kubectl get pods -n ${namespace} -l ${labelSelector} --field-selector=status.phase=Running -o jsonpath='{.items[*].metadata.name}'", returnStdout: true).trim().split()

                    // Iterate over each running pod
                    for (pod in runningPods) {
                        // Check if the pod is actually not running
                        def execOutput = sh(script: "kubectl exec ${pod} -n ${namespace} -- bin/bash 2>&1", returnStatus: true)

                        // If the exec command failed with the expected error message, delete the pod
                        if (execOutput != 0 && execOutput.stderr.contains("cannot exec in a stopped container: unknown")) {
                            // Use kubectl to force delete the pod
                            sh "kubectl delete pod ${pod} -n ${namespace} --force --grace-period=0"
                        }
                    }
                }
            }
        }
    }
}

//sh script

@Library("com.optum.jenkins.pipeline.library@master") _

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
                                execOutput=\$(kubectl exec \${pod} -n ${k8sNamespace} -- bin/bash 2>&1)
            
                                if [ \$? -ne 0 ] && echo "\${execOutput}" | grep -q "cannot exec in a stopped container: unknown"; then
                                  # kubectl delete pod \${pod} -n ${k8sNamespace} --force --grace-period=0
                                  echo Delete
                                fi
                            done
                        """
                    }
                }

            }
        }

    }
}

