pipeline {
    agent any
	
	environment {
        adserviceRegistry = "mrolu.dev:5000/micros/adservice"
        cartserviceRegistry = "mrolu.dev:5000/micros/cartservice"
        mainRegistry = "repository.k8sengineers.com/apexrepo/main"
        registryCredential = 'mrolu'
        myRegistry = "https://mrolu.dev:5000"
    }
	
	stages {
	
	  stage('Build adservice Image') {
        when { changeset "src/adservice/*"}
	     steps {
		   
		     script {
                dockerImage = docker.build( adserviceRegistry + ":$BUILD_NUMBER", "./src/adservice")
             }

		 }
	  
	  }
	  
	  stage('Deploy adservice Image') {
          when { changeset "src/adservice/*"}
          steps{
            script {
              docker.withRegistry( myRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
	   }

       stage('Kubernetes Deploy adservice') {
           when { changeset "client/*"}
            steps {
                  withCredentials([file(credentialsId: 'CartWheelKubeConfig1', variable: 'config')]){
                    sh """
                      export KUBECONFIG=\${config}
                      pwd
                      helm upgrade kubekart kkartchart --install --set "kkartcharts-frontend.image.client.tag=${BUILD_NUMBER}" --namespace kart
                      """
                  }
                 }  
        }

        stage('Build books Image') {
        when { changeset "javaapi/*"}
	     steps {
		   
		     script {
                dockerImage = docker.build( booksRegistry + ":$BUILD_NUMBER", "./javaapi/")
             }

		 }
	  
	  }
	  
	  stage('Deploy books Image') {
          when { changeset "javaapi/*"}
          steps{
            script {
              docker.withRegistry( cartRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
	   }

       stage('Kubernetes books Deploy') {
           when { changeset "javaapi/*"}
            steps {
                  withCredentials([file(credentialsId: 'CartWheelKubeConfig1', variable: 'config')]){
                    sh """
                      export KUBECONFIG=\${config}
                      pwd
                      helm upgrade kubekart kkartchart --install --set "kkartcharts-backend.image.books.tag=${BUILD_NUMBER}" --namespace kart
                      """
                  }
                 }  
        }

        stage('Build Main Image') {
        when { changeset "nodeapi/*"}
	     steps {
		   
		     script {
                sh " sed -i 's/localhost/emongo/g' nodeapi/config/keys.js"
                dockerImage = docker.build( mainRegistry + ":$BUILD_NUMBER", "./nodeapi/")
             }

		 }
	  
	  }
	  
	  stage('Deploy Main Image') {
          when { changeset "nodeapi/*"}
          steps{
            script {
              docker.withRegistry( cartRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
	   }

       stage('Kubernetes Main Deploy') {
           when { changeset "nodeapi/*"}
            steps {
                  withCredentials([file(credentialsId: 'CartWheelKubeConfig1', variable: 'config')]){
                    sh """
                      export KUBECONFIG=\${config}
                      pwd
                      helm upgrade kubekart kkartchart --install --set "kkartcharts-backend.image.main.tag=${BUILD_NUMBER}" --namespace kart
                      """
                  }
                 }  
        }
	}
}