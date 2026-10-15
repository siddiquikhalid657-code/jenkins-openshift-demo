
pipeline {
    agent any

    environment {
        OCP_SERVER = 'https://api.crc.testing:6443'
        OCP_PROJECT = 'jenkins-demo'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/siddiquikhalid657-code/jenkins-openshift-demo.git'
            }
        }

        stage('OpenShift Login') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'openshift-token',
                        variable: 'OCP_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        oc login "$OCP_SERVER" \
                          --token="$OCP_TOKEN" \
                          --insecure-skip-tls-verify=true
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    oc project "$OCP_PROJECT"

                    oc create configmap nginx-demo-html \
                      --from-file=index.html \
                      --dry-run=client -o yaml |
                      oc apply -f -

                    oc apply -f deployment.yaml

                    oc rollout status deployment/nginx-demo \
                      -n "$OCP_PROJECT" --timeout=180s
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    oc get pods -n "$OCP_PROJECT"
                    oc get svc nginx-demo -n "$OCP_PROJECT"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed to OpenShift!'
        }
        failure {
            echo 'FAILED: Check Console Output for the error.'
        }
    }
}
