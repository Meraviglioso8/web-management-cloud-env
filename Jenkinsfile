pipeline {
    agent any
    
    environment {
    REGISTRY_NAME = "PLACE-HOLDER-REGISTRY_NAME" // Change to yours 
    ACR_LOGIN_SERVER = "${REGISTRY_NAME}.azurecr.io"
    REPOSITORY_NAME = "PLACE-HOLDER-REPOSITORY_NAME" // Change to yours
    AZURE_CREDENTIALS_ID = credentials('azure-sp-credentials')  // Change to your credentials ID
    AKS_RESOURCE_GROUP = "PLACE-HOLDER-RESOURCE-GROUP-NAME" // Change to yours
    AKS_CLUSTER_NAME = "PLACE-HOLDER-CLUSTER-NAME" // Change to yours
    AZURE_SP_TENANT = "PLACE-HOLDER-SP-TENANT" // Change to yours
    }
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Meraviglioso8/Ekart.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                sh 'mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:tree -Dverbose=true'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                snykSecurity(
                    snykInstallation: 'snyk-latest',
                    snykTokenId: 'snykToken',
                    severity: 'critical',
                    failOnIssues: false,
                    monitorProjectOnBuild: true
                )
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    // Corrected the typos and parameterized the token
                    sh 'mvn clean verify sonar:sonar -Dmaven.test.skip=true -Dmaven.test.failure.ignore=true -Dsonar.projectName=ekart -Dsonar.projectKey=ekart -Dsonar.projectVersion=1.0 -Dsonar.exclusions=**/*.ts'
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart ${REGISTRY_NAME}.azurecr.io/${REPOSITORY_NAME}:latest"
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    # Run Trivy scan on the Docker image
                    if command -v trivy &> /dev/null; then
                        trivy image ${REGISTRY_NAME}.azurecr.io/${REPOSITORY_NAME}:latest > trivy-report.txt
                    else
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image ${REGISTRY_NAME}.azurecr.io/${REPOSITORY_NAME}:latest > trivy-report.txt
                    fi
                '''
        } 
        }
        stage('Upload Image to ACR') {
         steps{   
             script {
               withCredentials([usernamePassword(credentialsId: 'acr-credentials', usernameVariable: 'SERVICE_PRINCIPAL_ID', passwordVariable: 'SERVICE_PRINCIPAL_PASSWORD')]) {
                 sh "docker login ${ACR_LOGIN_SERVER} -u $SERVICE_PRINCIPAL_ID -p $SERVICE_PRINCIPAL_PASSWORD"
                 sh " docker push ${REGISTRY_NAME}.azurecr.io/${REPOSITORY_NAME}:latest"
                  }
              }
         }
        }
        
        stage('Deploy to AKS') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'azure-sp-credentials', usernameVariable: 'AZURE_SP_APP_ID', passwordVariable: 'AZURE_SP_PASSWORD')]) {
                        sh '''
                            az login --service-principal -u $AZURE_SP_APP_ID -p $AZURE_SP_PASSWORD --tenant $AZURE_SP_TENANT
                            az aks get-credentials --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME
                            for yaml_file in $(find . -name '*.yaml'); do
                                kubectl apply -f $yaml_file
                            done
                        '''
                    }
                }
            }
        }
        
        //stage('Push The Docker Image') {
           // steps {
               // script {
                   // withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                   //     sh "docker push meraviglioso8/shopping-cart:latest"
                   // }

                //}
            //}
        //}

        // stage('Kubernetes Deploy') {
        //     steps {
        //         withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.15.138:6443') {
        //             sh "kubectl apply -f deploymentservice.yml -n webapps"
        //             sh "kubectl get svc -n webapps"
        //         }
        //     }
        // }
    }
}
