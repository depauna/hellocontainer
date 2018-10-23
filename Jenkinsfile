podTemplate(label: 'buildpod',
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
    ],
    containers: [
        containerTemplate(name: 'docker', image: 'lachlanevenson/docker-make', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v2.7.2', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.10.9', command: 'cat', ttyEnabled: true)
  ]) {

    node('buildpod') {
        checkout scm
        container('docker') {
            stage('Build Docker Image') {
                sh """
                #!/bin/bash
                NAMESPACE=default
                REGISTRY=mycluster.icp:8500
                docker build -t \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER} .
                """
            } 
            stage('Push Docker Image to Registry') {
                sh """
                #!/bin/bash
                NAMESPACE=default
                REGISTRY=mycluster.icp:8500

                set +x
                DOCKER_USER=admin
                DOCKER_PASSWORD=passw0rd
                docker login -u=\${DOCKER_USER} -p=\${DOCKER_PASSWORD} \${REGISTRY}
                set -x

                docker push \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER}
                """
            }
        }
        container('kubectl') {
            stage('Test kubectl') {
                sh """
                #!/bin/bash
                set +e
                kubectl get nodes
                """
            }
        }
        container('helm') {
            stage('Deploy new helm release') {
                sh """
                #!/bin/bash
                set +e
                NAMESPACE=admin
                REGISTRY=passw0rd
                CHARTNAME=`helm list --deployed --short hello-container`

                helm list \${CHARTNAME}

                # Update Release 
                helm upgrade --wait --install --tls hello-container ./hellocontainer-chart/ --set image.repository=\${REGISTRY}/\${NAMESPACE}/hello-container --set image.tag=${env.BUILD_NUMBER}
                """
            }
        }


    }
}
