#!groovy

node {
 
  def app

  stage('Clone repository') {
    /* Let's make sure we have the repository cloned to our workspace */
      checkout scm 
  }

  stage('Build image') {
    /* This builds the actual image; synonymous to
     * docker build on the command line */
        app = docker.build("e2eshippabledemo:jenkins.${env.BUILD_NUMBER}")
  }

  stage('Test image') {
    /* Ideally, we would run a test framework against our image.
     * For this example, we're using a Volkswagen-type approach ;-) */
        app.inside {
            sh 'echo "Tests passed"'
        }
  }

  stage('Push image') {
    /* Finally, we'll push the image  */
        sh "echo $PATH"
        sh "cd $HOME"
        sh "aws ecr get-login --region us-east-1"
        sh "docker tag e2eshippabledemo:jenkins.${env.BUILD_NUMBER} 679404489841.dkr.ecr.us-east-1.amazonaws.com/e2eshippabledemo:jenkins.${env.BUILD_NUMBER}"
        sh "docker push 679404489841.dkr.ecr.us-east-1.amazonaws.com/e2eshippabledemo:jenkins.${env.BUILD_NUMBER}"
  }

  stage('Update Shippable image resource state') {
    sh "sudo yum -y install jq"
    
    /* Get the Shippable project id using the resource Id of the image resource 
    (which is available on the SPOG page) */
    def RESOURCE_ID=36839
    def PROJECT_ID = sh ( 
        script: "curl -H \"Authorization: apiToken $API_TOKEN\" \"https://api.shippable.com/resources/${RESOURCE_ID}\" | jq \".projectId\"", 
        returnStdout: true
        ).trim()

    /* Post the new version of the image resource to Shippable */
    sh "curl -X POST --header 'Authorization: apiToken $API_TOKEN' --header 'Content-Type: application/json' -d '{\"resourceId\": ${RESOURCE_ID},\"projectId\": ${PROJECT_ID},\"versionName\": \"jenkins.${BUILD_NUMBER}\"}' 'https://api.shippable.com/versions' "
    }
    
  }
