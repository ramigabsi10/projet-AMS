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
            SONAR_TOKEN_REST = '87cd7f7c09672196b840b2997a0da66b010e9220'       
            KUBECONFIG = credentials('kubeconfig-minikube') 
           }




          stages{

           //Build de projet (Spring)
           stage('Build  Projet spring') {
           steps {
           dir(env.BACKEND_DIR) {
            bat 'mvn clean install -DskipTests'
               }
             }
          }



        //SonarQube analysis for Backend (Spring)
        stage('SonarQube Analysis - Spring') {
            steps {
                dir(env.BACKEND_DIR) {
                    bat """
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
                    bat """
                        npm install
                        npm run sonar
                    """
                }
            }
        }



        // Création Image docker et push vers dockerhub backend
        stage('Creation d\'une image backend - push vers dockerhub') {
            steps {
                bat 'docker build -t ramidokub/amsrest2024 ${BACKEND_DIR}'

                bat 'echo $DOCKERHUB_CREDENTIALS_PSW \
                  | docker login -u $DOCKERHUB_CREDENTIALS_USR \
                  --password-stdin'

                bat 'docker push ramidokub/amsrest2024'
            }

                post {

                  always {

                  bat 'docker logout'

                         }

                     }
        }


        // Création Image docker et push vers dockerhub frontend
        stage('Creation d\'une image frontend - push vers dockerhub') {
            steps {
                bat 'docker build -t ramidokub/amsfront2024 ${FRONTEND_DIR}'

                bat 'echo $DOCKERHUB_CREDENTIALS_PSW \
                  | docker login -u $DOCKERHUB_CREDENTIALS_USR \
                  --password-stdin'

                bat 'docker push ramidokub/amsfront2024'
            }

                post {

                  always {

                  bat 'docker logout'

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
                        bat 'helm upgrade --install ams-app ./helm-chart-ams'
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



