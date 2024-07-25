pipeline {
  
          agent any
		  
		  tools {
           maven 'maven384'
           nodejs "nodejs20100"
          }

          environment {
            DOCKERHUB_CREDENTIALS = credentials('dockerhub_cred')
            BACKEND_DIR = 'amsRest2024'
            FRONTEND_DIR = 'amsFront2024'
            SONAR_HOST_URL = 'http://localhost:9000'
            SONAR_TOKEN_REST = 'd6e47c461e172bfbdec18680ee6de233843170c1'       
            KUBECONFIG = credentials('kubeconfig-minikube') 
           }




          stages{

           //Build de projet (Spring)
           stage('Build  Projet spring') {
           steps {
           dir(env.BACKEND_DIR) {
            sh 'mvn clean install -DskipTests'
               }
             }
          }



        //SonarQube analysis for Backend (Spring)
        stage('SonarQube Analysis - Spring') {
            steps {
                dir(env.BACKEND_DIR) {
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=amsRest2024 \
                            -Dsonar.host.url=${env.SONAR_HOST_URL} \
                            -Dsonar.login=${env.SONAR_TOKEN_REST} 
                    """
                }
            }
        }

        //SonarQube analysis for Frontend (Angular)
        stage('SonarQube Analysis - Angular') {
            steps {
                dir(env.FRONTEND_DIR) {
                    sh """
                        npm install
                        npm run sonar
                    """
                }
            }
        }


        // Upload the .jar file to Nexus repository
        stage('Upload jar to nexus') {
            steps {
                dir(env.BACKEND_DIR) {
                    script {
                        nexusArtifactUploader artifacts: [
                            [
                                artifactId: 'ams_rest',
                                classifier: '',
                                file: 'target/ams_rest-0.0.1-SNAPSHOT.jar',
                                type: 'jar'
                            ]
                        ],
                        credentialsId: 'nexus_cred',
                        groupId: 'com.sip',
                        nexusUrl: 'localhost:8081',
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        repository: 'ams_rest',
                        version: '0.0.1-SNAPSHOT'
                    }
                }
            }
        }

        // Création Image docker et push vers dockerhub backend
        stage('Creation d\'une image backend - push vers dockerhub') {
            steps {
                sh 'docker build -t ramidokub/amsrest2024 ${BACKEND_DIR}'

                sh 'echo $DOCKERHUB_CREDENTIALS_PSW \
                  | docker login -u $DOCKERHUB_CREDENTIALS_USR \
                  --password-stdin'

                sh 'docker push ramidokub/amsrest2024'
            }

                post {

                  always {

                  sh 'docker logout'

                         }

                     }
        }


        // Création Image docker et push vers dockerhub frontend
        stage('Creation d\'une image frontend - push vers dockerhub') {
            steps {
                sh 'docker build -t ramidokub/amsfront2024 ${FRONTEND_DIR}'

                sh 'echo $DOCKERHUB_CREDENTIALS_PSW \
                  | docker login -u $DOCKERHUB_CREDENTIALS_USR \
                  --password-stdin'

                sh 'docker push ramidokub/amsfront2024'
            }

                post {

                  always {

                  sh 'docker logout'

                         }

                     }

        } 

          //Déploiement de helm chart sur minikube k8s
          stage('Deploy Helm Chart') {
            steps {
                script {
                    withCredentials([
                        file(credentialsId: 'kubeconfig-minikube', variable: 'KUBECONFIG')
                    ]) {
                        sh 'helm upgrade --install ams-app ./helm-chart-ams'
                    }
                }
              }
            }
   
   
    }

          // Mail notification
          post {
                      
              always {

                  script {
                      //def toAddresses = "ttt@gmail.com,,ttt2@gmail.com"
                      def toAddresses = "rami.gabsi5@gmail.com"
                      def subject = "jenkins build:${currentBuild.currentResult}-${BUILD_NUMBER}: ${env.JOB_NAME}"
                      def body = "${currentBuild.currentResult}: Job ${env.JOB_NAME}.Check the attachement to view the details."
                            
                            emailext (
                              to: toAddresses,
                              subject: subject,
                              body: body,
                              replyTo: "rami.gabsi5@gmail.com",
                              mimeType: 'text/html',
                              attachLog: true,
                                    )

                            }
                      }

            }  

     }



