podTemplate(label: 'buildpod',
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        hostPathVolume(hostPath: '/root/.helmtest', mountPath: '/etc/helmtest')
    ],
    containers: [
        containerTemplate(name: 'docker', image: 'mycluster.icp:8500/default/k8s-docker:1', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'helm', image: 'mycluster.icp:8500/default/k8s-helm:2.7.2', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'kubectl', image: 'mycluster.icp:8500/default/k8s-kubectl:1.10.9', command: 'cat', ttyEnabled: true)
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

                DOCKER_USER=admin
                DOCKER_PASSWORD=passw0rd
                docker login -u=\${DOCKER_USER} -p=\${DOCKER_PASSWORD} \${REGISTRY}

                docker push \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER}
                """
            }
        }
        container('kubectl') {
            stage('Test kubectl') {
                sh """
                #!/bin/bash
                kubectl get nodes
                """
            }
        }
        container('helm') {
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
}
