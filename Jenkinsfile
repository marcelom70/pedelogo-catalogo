pipeline{
    agent any
    stages{
        stage("Build image"){
            steps{
                script{
                    dockerapp = docker.build("marcelom70/api.produto:${env.BUILD_ID}", 
                        "-f ./src/PedeLogo.Catalogo.Api/Dockerfile .")
                }
            }
        }

        stage("Push image"){
            steps{
                script{
                    docker.withRegistry("https://registry.hub.docker.com", "hub.docker.id"){
                        dockerapp.push("latest")
                        dockerapp.push("${env.BUILD_ID}")
                    }
                    
                }
            }
        }

        stage("Deploy kubernetes"){
            agent{
                kubernetes{
                    cloud 'kubernetes'
                }
            }
            environment{
                tag_version = "${env.BUILD_ID}"
            }
            steps{
                withKubeConfig([credentialsId: 'kubernetes']) {
                        sh 'sed -i "s/{{tag}}/$tag_version/g" ./Manifestos/api/deployment.yaml'
                        sh 'getent hosts $HOSTNAME'
                        sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
                        sh 'chmod +x kubectl'
                        sh './kubectl apply -f ./Manifestos/mongodb/'
                        sh './kubectl apply -f ./Manifestos/api/'
                }         
            }
        }
    }
}
