node {
   // Mark the code checkout 'stage'....
   stage 'checkout'

   // Get some code from a GitHub repository
   git url: 'https://github.com/gleidsoncampos/pipeline_test/'
   sh 'git clean -fdx; sleep 4;'

   // Get the maven tool.
   // ** NOTE: This 'mvn' maven tool must be configured
   // **       in the global configuration.

   stage 'build'
   // set the version of the build artifact to the Jenkins BUILD_NUMBER so you can
   // map artifacts to Jenkins builds
   sh "echo \"*******-Starting CI CD Pipeline Tasks-*******\""
   sh "echo \"\""
   sh "echo \"..... Build Phase Started :: Compiling Source Code :: ......\""
   //sh "cd /root/.jenkins/workspace/teste-pipe/java_web_code"
   //sh "sleep 6;"
   sh "mvn install -f  /root/.jenkins/workspace/teste-pipe/java_web_code/pom.xml"

   stage 'test'
   parallel 'test': {
     sh "echo \"\""
     sh "echo \"..... Test Phase Started :: Testing via Automated Scripts :: ......\""
     //sh "cd ../integration-testing/"
     sh "mvn clean verify -f /root/.jenkins/workspace/teste-pipe/integration-testing/pom.xml"
   }, 'verify': {
       sh 'echo \"write your test code here\"; sleep 6;'
   }

   stage 'archive'
   archive 'target/*.jar'
}


node {
   def CONTAINER= "devops_pipeline_demo"
   stage 'deploy Canary'
   sh "echo \"\""
   sh "echo \"..... Integration Phase Started :: Copying Artifacts :: ......\""
   //sh "cd java_web_code/"
   sh "/bin/cp /root/.jenkins/workspace/teste-pipe/java_web_code/target/wildfly-spring-boot-sample-1.0.0.war /root/.jenkins/workspace/teste-pipe/docker/"
   sh "echo \"\""
   sh "echo \"..... Provisioning Phase Started :: Building Docker Container :: ......\""
   //sh "cd ../docker/"
   sh "sudo docker build -t " +CONTAINER+ " /root/.jenkins/workspace/teste-pipe/docker/"


 
   RUNNING= sh (
       script: "sudo docker inspect --format=\"{{ .State.Running }}\" "+CONTAINER + "" || true, 
       returnStdout: true
    ).trim()
   
   if (RUNNING == "true") {
       sh ("sudo docker rm -f "+CONTAINER)
   } 

    //run your container
    sh "echo \"\""
	sh "echo \"..... Deployment Phase Started :: Building Docker Container :: ......\""
	sh "sudo docker run -d -p 8180:8080 --name " +CONTAINER + " "+CONTAINER 


//#-Completion
    sh "echo \"--------------------------------------------------------\""
    sh "echo \"View App deployed here: http://server-ip:8180/sample.txt\""
    sh "echo \"--------------------------------------------------------\""

   stage 'deploy Production'
   input 'Proceed?'
   sh 'echo \"write your deploy code here\"; sleep 6;'
   archive 'target/*.jar'
}