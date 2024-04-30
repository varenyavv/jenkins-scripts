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

#!/bin/bash

# Define Kubernetes namespace and label selector for identifying pods to clean up
namespace="your-namespace"
labelSelector="kubernetes_pod_operator=True"

# Get list of running pods with the specified label selector in the specified namespace
runningPods=$(kubectl get pods -n ${namespace} -l ${labelSelector} --field-selector=status.phase=Running -o jsonpath='{.items[*].metadata.name}' 2>/dev/null)

# Iterate over each running pod
for pod in ${runningPods}; do
    # Check if the pod is actually not running
    execOutput=$(kubectl exec ${pod} -n ${namespace} -- bin/bash 2>&1)
    
    # If the exec command failed with the expected error message, delete the pod
    if [ $? -ne 0 ] && echo "${execOutput}" | grep -q "cannot exec in a stopped container: unknown"; then
        # Force delete the pod
        kubectl delete pod ${pod} -n ${namespace} --force --grace-period=0
    fi
done

