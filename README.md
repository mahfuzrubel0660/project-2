# project-2



Project 2: Implementation.
Deploying web app using Jenkins CI/CD declarative pipeline.
Follow the steps:

1.     First of all, go to the AWS portal, and create a new instance. As
·       Name: jenkins-server
·       AMI: ubuntu.
·       Instance type: t2.micro (free tier).
·       Key pair login: Create > docker.pem.
·       Allow HTTP.
·       Allow HTTPS.
·       (Download the .pem file.)
·       Click on Launch Instance.
 
 
2.    Now, we will allow ports 8080 and 8001 for this machine from a security group. We can find the security group in the VM description. Now, here we need to allow “Inbound Rule”

3.     Now, connect to the EC2 instance that you have created. Copy the SSH from server:
 

4.     Go to the download folder, where the .pem file is placed and open the terminal in the same location, and paste the SSH.

5.     Install Jenkins from following link:
https://www.jenkins.io/doc/book/installing/linux/

6.     Install Docker by running:
“Sudo apt-install docker.io”

7.     Now check if it got installed by running “jenkins --version” and “docker --version”
 

8.     Goto Jenkins Dashboard and Click on “New Item”
·       Name: declarative_pipeline
·       Select: Pipeline
Description: This is a demo pipeline project.

9.     Pipeline Script:

    
pipeline{
    environment {
        DOCKER_HUB_USERNAME = 'mahfuzrubel0660'
        DOCKER_HUB_PASSWORD = 'dckr_pat_c_RqK-lkXbJ7CplIWuoX_L0Zc-s'
        IMAGE_NAME = 'mahfuzrubel0660/app2'
        IMAGE_TAG = 'latest'
    }
    agent any
    stages{
        stage("clone"){
            steps{
                git (
                    branch: "main", 
                    url: 'https://github.com/mahfuzrubel0660/project-2.git'
                )
            }
        }
        stage("testing"){
            steps{
                echo "Testing"
            }
        }
        stage("build the image"){
            steps{
                script{
                    sh "docker build -t mahfuzrubel0660/app2 ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub
                    sh "docker login -u ${DOCKER_HUB_USERNAME} -p ${DOCKER_HUB_PASSWORD}"

                    // Tag the built image
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"

                    // Push the tagged image to Docker Hub
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        
        stage("stop the container"){
            steps{
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE'){
                    script{
                        sh "docker stop app2"
                    }
                }
                
            }
        }
        stage("remove the container"){
            steps{
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE'){
                    script{
                        sh "docker rm app2"

                    }
                }
                    
            }
        }
        
        stage("run the docker container"){
            steps{
                script{
                    sh "docker run --name app2 -p 8002:80 -d mahfuzrubel0660/app2"
                }
            }
        }
        
    }
} 



 

10.  Click on “Save”.

11.  Click on “Build Now”.

All project run on /var/jenkins_home/workspace as we build the jenkin with this direction.
docker run -it -d -u root -p 8080:8080 -p 50000:50000 -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker -v /var/jenkins_home:/var/jenkins_home --name jenkins jenkins



 

12.  We can click on “Stage View” to check the results if it is still under process.
 

13.  Once it will be completed, It will return as “Success”.
 

14.  Now, Search for the Public IP with port 8001 in the browser,
 

We have built the image of a Web app source code, and deployed that image as a container, using Jenkins Pipelines.

Now, our goal is,
·       Whenever the developer commits their code in GitHub, after every commit, it should reflect in the live web app.
·       For that, we will use “GitScm polling”.
·       Every time, a developer made a commit, a trigger will run automatically, which will rebuild the image and run a container on your behalf as a part of automation that will run the pipeline automatically.

25.  Now, configure the project again, and add
Build Trigger: GitHub hook trigger for GitScm polling.
Description: GitHub webhook integration
 

26.  We need to install the “Git Integration” plugin from Manage Jenkins, by following the path,
(Manage Jenkins > Manage Plugins > Git Integration).
 
 
27.  Now, Goto GitHub > Settings > SSH and GPG Keys > New SSH Key.
Add details as
Title: chetanrakhra@gmail.com
Key type: <public_key>
 
 
28.  To get the Public key, open the “id_rsa.pub” file and copy the content. (Public Key) 
Of the jenkin machine.
 

29.  Now, we need to go to GitHub and create a new SSH and GPG Key.
GitHub > Repo “react-django-demo-app” > Settings > Webhooks.
30.  Add the following details,
Payload URL: http://<public_ip_of_ec2>:8080/github-webhook/
Content Type: application/json
Which event would you like to trigger this webhook?
o  Just the push event.
Active: True
Click on “Add Webhook”.
 
 
31.  Now, Save the configured project.
32.  Do some changes in the code and push to GitHub, this will automatically run a pipeline, and the new code will be Live.


Git command:
git pull https://github.com/mahfuzrubel0660/project-1.git
git add .
git commit -m "forth commit"
git push -u origin main





