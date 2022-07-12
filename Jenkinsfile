node {
    // reference to maven
    // ** NOTE: This 'maven-3.6.1' Maven tool must be configured in the Jenkins Global Configuration.   
    def mvnHome = tool 'maven-3.8.5'

    // holds reference to docker image
    def dockerImage
    // ip address of the docker private repository(nexus)
    
    //def dockerRepoUrl = "localhost:8083"
    def dockerImageName = "yasantha1995/springboot-app"
    def dockerImageTag = "${dockerImageName}:${env.BUILD_NUMBER}"
    
    stage('Clone Repo') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/yasa1995/hello-world-app-springboot.git'
      // Get the Maven tool.
      // ** NOTE: This 'maven-3.6.1' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'maven-3.8.5'
    }    
  
    stage('Build Project') {
      // build project via maven
      sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
    }
	
	stage('Publish Tests Results'){
      parallel(
        publishJunitTestsResultsToJenkins: {
          echo "Publish junit Tests Results"
		  junit '**/target/surefire-reports/TEST-*.xml'
		  archive 'target/*.jar'
        },
        publishJunitTestsResultsToSonar: {
          echo "This is branch b"
      })
    }
		
    stage('Build Docker Image') {
      // build docker image
      sh "whoami"
      sh "ls -all /var/run/docker.sock"
      sh "mv ./target/hello*.jar ./data" 
      
      dockerImage = docker.build("yasantha1995/springboot-app")
    }
   
    stage('Deploy Docker Image'){
      
      // deploy docker image to nexus

      echo "Docker Image Tag Name: ${dockerImageTag}"

      //sh "docker login -u admin -p admin123 ${dockerRepoUrl}"
      sh "docker tag ${dockerImageName} ${dockerImageTag}"
      sh "docker push ${dockerImageTag}"
    }

    stage("vm provisioning"){
      environment {
//        TF_VAR_env_prefix = 'prod'
      }
        script {
          dir('terraform') {
            sh "terraform init"
            sh "terraform apply -var env_prefix=prod -auto-approve"
            SVG_IP = sh (
              script: "terraform output vm-public-ip",
              returnStdout: true
              ).trim()
          }
      }
          

    }

    stage ("Deploy") {
      script {
        echo "waiting for server initizalition"
        sleep(time:90, unit:'SECONDS')
        echo "${SVG_IP}"
        echo "Deploying docker image to production server"
        def production_server = "yasantha@${SVG_IP}"
        echo "${production_server}"

        sshagent (credentials: ['ssh-key-deploy']){
          sh 'ssh -o StrictHostKeyChecking=no yasantha@${SVG_IP} docker run -d -p 8080:8080 -it --rm yasantha1995/springboot-app'
          echo "webserver is hosted on http://${SVG_IP}:8080"

        }
      }
    }
    
}
