pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "Files in workspace:"
                    ls -la
                '''
            }
        }

        stage('OpenShift Login') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'openshift-token',
                        variable: 'OPENSHIFT_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x

                        oc login https://172.16.250.1:6443 \
                        --token="$OPENSHIFT_TOKEN" \
                        --insecure-skip-tls-verify=true

                        oc project jenkins-demo
                    '''
                }
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                sh '''
                    oc apply -f deployment.yaml
                '''
            }
        }

        stage('Check Deployment') {
            steps {
                sh '''
                    echo "Checking deployment..."

                    echo "Deployment:"
                    oc get deployment nginx-demo

                    echo "Pods:"
                    oc get pods

                    echo "Service:"
                    oc get service nginx-demo

                    echo "Route:"
                    oc get route nginx-demo
                '''
            }
        }

        stage('Deployment Status') {
            steps {
                sh '''
                    oc rollout status deployment/nginx-demo
                '''
            }
        }
    }

    post {

        success {
            echo 'Application deployed successfully!'
        }

        failure {
            echo 'Deployment failed!'
        }
    }
}
