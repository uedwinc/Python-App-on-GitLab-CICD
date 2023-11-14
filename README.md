# Building Python App on Gitlab CICD

![Gitlab-CICD.png](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/GitLab-CICD.png)

This is a simple Python Flask web application. The app provides system information and a realtime monitoring screen with dials showing CPU, memory, IO and process information.
The python application is written as a [makefile](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/makefile)

The application files for this project is cloned from https://github.com/seunayolu/gitlab-project (Original project archived at https://github.com/benc-uk/python-demoapp/)

This project involves:

- Testing, packaging and building a python application into a docker image
- Running the image in an EC2 instance on AWS
- Configuring slack notification

## Configure slack notification

- Under any particular project, go to Settings > Integrations
    - Search 'GitLab for Slack' in the search box
    - Locate the GitLab for Slack app and click 'Configure'
    - Click 'install GitLab for Slack app...'
    - This opens up slack sign in to your workspace
    - Click 'find your workspaces'
    - Sign in and select any slack workspace
    - Click 'Allow'
    - Confirm successful installation
        - Scroll down to 'Triggers'
            - Check 'A push is made to the repository' and specify channel eg #general
            - Check 'A pipeline status changes' and specify channel eg #general
            - Check 'A deployment is started or finished' and specify channel eg #general
        - Scroll down to 'Notification settings'
            - Uncheck 'Notify only broken pipelines'
            - Select 'All branches' for notifications to be sent
            - For 'Labels to be notified behavior', select 'Match all of the labels'
        - Save changes and confirm connection successful

NB:
- You will get an email notification in case of any pipeline failure and if the pipeline is fixed

## CICD Process

1. Run Test

- The tests are specified in the makefile by the programmers

2. Build Image

- Inorder to configure docker login:
    - Go to your project on Gitlab
    - Under Settings, click 'CI/CD'
    - Go to variables and 'Add variable'
        - Check 'Masked'
        - key = USERNAME
        - value = your-username (that is, dockerhub username)
    - Add variable again
        - Check 'Masked'
        - key = PASSWORD
        - value = your-password (that is, dockerhub password)

- For docker build to push to docker hub:
    - Go to docker hub and create a repository
    - Give it a name (eg pythonapp)
    - Make it private and create

![pythonapp repo](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/images/pythonapp%20repo.png)

- Confirm on dockerhub if image was successfully pushed after running the pipeline

![pushed](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/images/pushed.png)

3. Deploying the image to EC2

- Start up an instance on AWS (ubuntu) and ssh into it
- Do `apt update`
- Install docker
    ```
    apt install docker.io -y
    ```
- Add the 'ubuntu' user to docker group
    ```
    usermod -aG docker ubuntu
    ```
- Add the instance key pair to the variables on gitlab
    - Under 'Add variable' type, select 'File'
    - No need to check any of the flags
    - key = SSH_KEY
    - value = content of the key pair.pem file

- Confirm deployment:

- `docker ps`

![docker ps](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/images/docker%20ps.png)

- Pipeline success

![passed](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/images/passed.png)

- Open port 5000 on the instance security group and open the application in a browser with ip address and port

![pythonapp](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/images/pythonapp.png)

- Slack Notification check

![slack](https://github.com/uedwinc/Python-App-on-GitLab-CICD/blob/main/images/slack.png)
