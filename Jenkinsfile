pipeline {
    agent any
    tools {
        nodejs "nodejs"
        terraform "terraform"
    }
    stages {
        stage("Build") {
            steps {
                script {
                    sh """
                    npm install
                    npm run build
                    """
                }
            }
        }
        stage("Test") {
            steps {
                script {
                    sh script: "npm test", returnStatus: true
                }
            }
        }
        stage("Deploy") {
            environment {
                ARTIFACT = sh (returnStdout: true, script: 
                """
                aws s3api list-buckets --query 'Buckets[].Name' | grep -wo "\\w*playgroundartifact\\w*"
                """
                ).trim()
                TFSTATE = sh (returnStdout: true, script: 
                """
                aws s3api list-buckets --query 'Buckets[].Name' | grep -wo "\\w*playgroundtfstate\\w*"
                """
                ).trim()
            }
            steps {
                script {
                    sh """
                    zip -r $UNIQUE_ANIMAL_IDENTIFIER-build-artifacts.zip build/
                    aws s3 cp $UNIQUE_ANIMAL_IDENTIFIER-build-artifacts.zip s3://${ARTIFACT}
                    cd terraform
                    terraform init -no-color -backend-config="key=${UNIQUE_ANIMAL_IDENTIFIER}.tfstate" -backend-config="bucket=${TFSTATE}"
                    terraform apply --auto-approve -no-color
                    """
                }
            }
        }
        stage("Show Domain") {
            steps {
                script {
                    sh script: "bash ${WORKSPACE}/scripts/display-dns.sh ${UNIQUE_ANIMAL_IDENTIFIER}", returnStatus: true
                }
            }
        }
        stage("destroy"){
            steps{
                script{
                    sh"""
                    
                    echo "terraform destroy --auto-approve"
                    """
                }
            }
        }
    }
    post {
        cleanup {
            deleteDir()
        }
    }
}
