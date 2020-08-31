/**pipeline{
    agent any 
	 tools {
        maven 'Maven-3.6.1'
    } 
     stages{
//Build the Docker Image based on the Dockerfile
       stage(" Maven Clean Package"){
	  steps{
	    sh "mvn -Dmaven.test.failure.ignore=true install"
	  }
       }
        stage('Build Docker Image'){
	  steps{
	     sh "sudo docker build -t maniengg/springboot1.2:${BUILD_ID} ."   //when we run docker in this step, we're running it via a shell on the docker build-pod container
                        }
       }
 // Pushing the Docker Image to DockerHub   
     stage('Push Docker Image'){
	steps{
//Docker Hub Credentials
        withCredentials([string(credentialsId: 'DOKCER_HUB_PASSWORD', variable: 'DOKCER_HUB_PASSWORD')]) {
          sh "sudo docker login -u maniengg -p ${DOKCER_HUB_PASSWORD}"
          sh "sudo docker push maniengg/springboot1.2:${BUILD_ID}"
	              }
           }
       }    
     stage("Deploy To Kuberates Cluster"){
      steps{
//Created Service Account in GCP console and generated key is added it to the Jenkins Credentials.
        withCredentials([file(credentialsId: 'demo-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
//activate the service account
         sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
//Configuring the project details to Jenkins and communicate with the gke cluster
         sh "gcloud config set project mssdevops-284216" //configuring name of the project
         sh "gcloud config set compute/zone us-central1-c"// confiuring the project compute-zone
         sh "gcloud config set compute/region us-central1"// confiuring the project compute-region
         sh "gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project mssdevops-284216"// command to connect the GKE cluster through command line

         sh "sed -i -e 's,image_to_be_deployed,'maniengg/springboot1.2:${BUILD_ID}',g' springBootMongo.yml"
         sh "kubectl apply -f springBootMongo.yml"
    }
}

     }
     }
}**/

node{
     
    stage('SCM Checkout'){
        git credentialsId: 'GIT_CREDENTIALSS', url: 'https://github.com/manikarnam/spring-boot-mongo-docker.git',branch: 'master'
    }
   
    stage(" Maven Clean Package"){
      def mavenHome =  tool name: "Maven-3.6.1", type: "maven"
      def mavenCMD = "${mavenHome}/bin/mvn"
      sh "${mavenCMD} clean package"
      
    } 
    
    
    stage('Build Docker Image'){
	    sh "sudo docker build -t maniengg/spring-boot-mongo:${BUILD_ID} ."
    }
    
    stage('Push Docker Image'){
        withCredentials([string(credentialsId: 'DOKCER_HUB_PASSWORD', variable: 'DOKCER_HUB_PASSWORD')]) {
          sh "sudo docker login -u maniengg -p ${DOKCER_HUB_PASSWORD}"
        }
        sh "sudo docker push maniengg/spring-boot-mongo:${BUILD_ID}"
     }
     
    /** stage("Deploy To Kuberates Cluster"){
       kubernetesDeploy(
         configs: 'springBootMongo.yml', 
         kubeconfigId: 'mykubeconfig',
         enableConfigSubstitution: true
        )
     }**/
	 
     stage("Deploy To Kuberates Cluster"){
        withCredentials([file(credentialsId: 'demo-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
         sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
         sh "gcloud config set project mssdevops-284216"
         sh "gcloud config set compute/zone us-central1-c"
         sh "gcloud config set compute/region us-central1"
         sh "gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project mssdevops-284216"
         sh "sed -i -e 's,image_to_be_deployed,'maniengg/spring-boot-mongo:${BUILD_ID}',g' springBootMongo.yml"
         sh "kubectl apply -f springBootMongo.yml"
        }
      }
}
