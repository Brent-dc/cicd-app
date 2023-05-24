# CICD



## 1.2 Build and verify the sample application
Making sample-app.sh executable
```
chmod u+x sample-app.sh
```
Running sample-app.sh will create another container with the application.
With the assigned ip of the dockerlab (via the yaml file) and the forwarded port 5050 should give 
 ```
 <h1>You are calling me from 192.168.56.1</h1>
```
Removing the SampleRunning docker
```
docker stop samplerunning
docker rm samplerunning
```

## 1.3&4 Download and run the Jenkins Docker image
Downloading image : 
```
  docker pull jenkins/jenkins:lts 
```
Start the docker via :
```
docker run -p 8080:8080 -u root \
  -v jenkins-data:/var/jenkins_home \
  -v $(which docker):/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  --name jenkins_server jenkins/jenkins:lts
 ```
With the given port that is forwarded 8080 in this case you can open the jenkins dashboard on your physical system
![Screenshot 2023-05-24 094501](https://github.com/Brent-dc/cicd-app/assets/61025631/70a0698d-9f5f-4b8b-929c-b8e580cac347)



## 1.5 Use Jenkins to build your application
Build a new freestyle job where you add a repository to pull from and a bash command to execute the shell script.
Before a build you shouldn't forget to remove the samplerunning container in the sample-app.sh script 
and add a line in the shell script to remove the tempdir folder in/var/jenkins_home/workspace/SampleApp
if the volume is persisted
![Screenshot 2023-05-24 094707](https://github.com/Brent-dc/cicd-app/assets/61025631/09379e0a-8f67-46a2-bca5-6c4e1c897d26)
Running the build should give this result without the TestSampleApp line at the end.
![Screenshot 2023-05-24 103523](https://github.com/Brent-dc/cicd-app/assets/61025631/bad1a6f7-fdb0-4907-84a0-dc3a308fe47a)



##  1.6 Add a job to test the application
A container's IP can be found with:
``` 
 docker inspect <container name>.
``` 
ex.
samplerunning: 172.17.0.2
jenkins_server: 172.17.0.3

Creating a new freestyle project that is built after SampleApp is built without errors, we can add a curl command to see if the server is running.
Building the SampleApp again shoud result in the above screenshot.


## 1.7 Create a build pipeline
We can combine the test and App in a pipeline via a new item => pipeline :
 ``` 
node {
    stage('Preparation') {
        catchError(buildResult: 'SUCCESS') {
            sh 'docker stop samplerunning'
            sh 'docker rm samplerunning'
        }
    }
    stage('Build') {
        build 'BuildSampleApp'
    }
    stage('Results') {
        build 'TestSampleApp'
    }
}
 ```
Building it should yield 

![Screenshot 2023-05-24 104133](https://github.com/Brent-dc/cicd-app/assets/61025631/f3db440a-158e-4c97-b40b-ba94cfa70fdf)
## 1.8 Create a build pipeline

Changing the colour of the background in the style.css file from the repo should also change the colour of the SampleApp when rebuilding.
