node('testnode') {
        checkout scm
            stage('Build Docker Image') {
                sh """
                #!/bin/bash
                NAMESPACE=default
                REGISTRY=mycluster.icp:8500
                docker build -t \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER} .
                """
            stage('Push Docker Image to Registry') {
                sh """
                #!/bin/bash
                NAMESPACE=default
                REGISTRY=mycluster.icp:8500

                DOCKER_USER=admin
                DOCKER_PASSWORD=passw0rd
                docker login -u=\${DOCKER_USER} -p=\${DOCKER_PASSWORD} \${REGISTRY}

                docker push \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER}
                """
            }
        }
            stage('Test kubectl') {
                sh """
                #!/bin/bash
                kubectl get nodes
                """
        }
            stage('Deploy new helm release') {
                sh """
                #!/bin/bash
                export HELM_HOME=/etc/helmtest
                NAMESPACE=default
                REGISTRY=mycluster.icp:8500
                CHARTNAME=`helm list --tls --deployed --short hello-container`

                helm list --tls \${CHARTNAME}

                # Update Release 
                helm upgrade --wait --install --tls hello-container ./hellocontainer-chart/ --set image.repository=\${REGISTRY}/\${NAMESPACE}/hello-container --set image.tag=${env.BUILD_NUMBER}
                """
            }


    }
}
