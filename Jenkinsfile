pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-token', url: 'https://github.com/SammekG/Capstone-DotNET-Mongo-CD.git'
            }
        }
        
        stage('Deploy To Kubernetes') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://A42FBEA650DC34F1A8B48F42C3875A03.gr7.ap-south-1.eks.amazonaws.com') {
                        sh 'kubectl apply -f Manifest/manifest.yaml -n webapps'
                        sh 'kubectl apply -f Manifest/ci.yaml'
                        sh 'kubectl apply -f Manifest/ingress.yaml -n webapps'
                        sleep 30
                    }
                }
            }
        }
        
         stage('Verify The Deployment') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://A42FBEA650DC34F1A8B48F42C3875A03.gr7.ap-south-1.eks.amazonaws.com') {
                        sh 'kubectl get pods -n webapps'
                        sh 'kubectl get svc -n webapps'
                        sh 'kubectl get ingress -n webapps'
                        sleep 30
                    }
                }
            }
        }
        
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'sammek.30@gmail.com',
                from: 'sammek.gandhi@gmail.com',
                replyTo: 'sammek.gandhi@gmail.com',
                mimeType: 'text/html',
               
            )
        }
    }
}
}
