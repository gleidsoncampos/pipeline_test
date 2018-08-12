node {
   // Mark the code checkout 'stage'....
   stage 'checkout'

   // Get some code from a GitHub repository
   git url: 'https://github.com/kesselborn/jenkinsfile'
   sh 'git clean -fdx; sleep 4;'

   // Get the maven tool.
   // ** NOTE: This 'mvn' maven tool must be configured
   // **       in the global configuration.
   def CONTAINER="devops_pipeline_demo"

   stage 'build'
   // set the version of the build artifact to the Jenkins BUILD_NUMBER so you can
   // map artifacts to Jenkins builds
   sh "echo \"*******-Starting CI CD Pipeline Tasks-*******\""
   sh "echo \"\""
   sh "echo \"..... Build Phase Started :: Compiling Source Code :: ......\""
   sh "cd java_web_code"
   sh "mvn install"

   stage 'test'
   parallel 'test': {
     sh "mvn test; sleep 2;"
   }, 'verify': {
     sh "echo \"\""
     sh "echo \"..... Test Phase Started :: Testing via Automated Scripts :: ......\""
     sh "cd ../integration-testing/"
     sh "mvn clean verify -P integration-test"
   }

   stage 'archive'
   archive 'target/*.jar'
}


node {
   stage 'deploy Canary'
   sh "echo \""
   sh "echo \"..... Integration Phase Started :: Copying Artifacts :: ......\""
   sh "cd java_web_code/"
   sh "/bin/cp target/wildfly-spring-boot-sample-1.0.0.war ../docker/"
   sh "echo \"\""
   sh "echo \"..... Provisioning Phase Started :: Building Docker Container :: ......\""
   sh "cd ../docker/"
   sh "sudo docker build -t devops_pipeline_demo ."


   //sh "CONTAINER=devops_pipeline_demo"
 
   sh "RUNNING=\$(sudo docker inspect --format=\"{{ .State.Running }}\" ${CONTAINER} 2> /dev/null)"

   sh "if [ $? -eq 1 ]; then"
     sh "echo \"\'${CONTAINER}\' does not exist.\""
   sh "else"
     sh "sudo docker rm -f ${CONTAINER}"
   sh "fi"

    //run your container
    sh "echo \"\""
	sh "echo \"..... Deployment Phase Started :: Building Docker Container :: ......\""
	sh "sudo docker run -d -p 8180:8080 --name devops_pipeline_demo devops_pipeline_demo"


//#-Completion
    sh "echo \"--------------------------------------------------------\""
    sh "echo \"View App deployed here: http://server-ip:8180/sample.txt\""
    sh "echo \"--------------------------------------------------------\""

   stage 'deploy Production'
   input 'Proceed?'
   sh 'echo "write your deploy code here"; sleep 6;'
   archive 'target/*.jar'
}