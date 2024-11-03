Update the Name tag on your Jenkins EC2 Instance to Jenkins-Ansible

(SCREENSHOT)

Install and Configure Ansible on EC2 Instance
Step 1: Connect to your EC2 Instance
Go to your AWS console, select EC2, and locate your instance for Jenkins-Ansible.
Copy the Public IP of your instance and use it to SSH into your instance:
bash
Copy code
ssh -i path_to_your_key.pem ubuntu@your_instance_public_ip

(Screenshot)

Once connected, update your instance:
bash
Copy code
sudo apt update

(screenshot)

Install Ansible
Install Ansible directly on the instance:
bash
Copy code
sudo apt install ansible -y

(screenshot)

Check your Ansible version by running ansible --version

(screenshot)

go to your GitHub account create a new repository and name it ansible-config-mgt.

(screenshot)

Configure Jenkins for Archiving Repository Content
Step 1: Create a New Freestyle Project in Jenkins
Open Jenkins (http://your_instance_public_ip:8080).

(screenshot)

Select New Item on the Jenkins dashboard.
Name your project ansible, select Freestyle project, and click OK.

(screenshot)

In the Source Code Management section, select Git.
In the Repository URL field, paste the URL of your ansible-config-mgt GitHub repository and your github crendentials

(screenshot)

allocate an Elastic IP to your Jenkins-Ansible server to avoid reconfiguring GitHub webhook to a new IP address

(sceenshot)

Set Up Webhook in GitHub to Trigger Jenkins Build
In GitHub, go to the Settings of your ansible-config-mgt repository.
Click on Webhooks > Add webhook.
In the Payload URL field, enter your Jenkins URL followed by /github-webhook/, for example:
plaintext
Copy code
http://your_instance_public_ip:8080/github-webhook/
Set Content type to application/json.
Under Which events would you like to trigger this webhook?, select Just the push event.
Click Add webhook to save.

(screenshot)

now go back tojenkins and click configure. scroll down to post-build actions

Configure Jenkins to Archive Build Artifacts
In the Post-build Actions section of your Jenkins job, select Archive the artifacts.
In the Files to archive field, type ** to archive all files.
mark or click on GitHub hook trigger for GITScm polling
Click Save to finish configuring your Jenkins job.

(screenshot)

Test the Setup
Open your ansible-config-mgt GitHub repository, and edit the README.md file. Make a minor change, such as adding a line.
Commit the changes directly to the master branch.

(screenshot)

In Jenkins, you should see that a build is triggered automatically after the commit.

(screenshot)

Once the build completes, verify that the build artifacts are archived in the following Jenkins directory:
bash
Copy code
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
Replace <build_number> with the specific build number.

(screenshot)

clone Repository to Jenkins-Ansible EC2 Instance
Back in your EC2 instance terminal, navigate to the directory where you want to clone the repository.
Clone your ansible-config-mgt repository:
bash
Copy code
git clone <repository-link>
Replace <repository-link> with the URL of your GitHub repository

(screenshot)

Prepare Your Development Environment with Visual Studio Code
Download Visual Studio Code from here and install it on your local machine.
After installing, open Visual Studio Code, and configure Git integration:
In Visual Studio Code, go to View > Terminal to open the terminal.
Navigate to the directory where you want to keep the repository on your local machine, and clone it:
bash
Copy code
git clone <repository-link>
After cloning, open the folder in Visual Studio Code and start editing your files directly in VS Code.

(screenshot)

Navigate to your GitHub Repository (ansible-config-mgt):

Open your repository on GitHub.
Click on Branches (found next to the “Code” tab).
Create a New Branch:

Checkout the New Branch Locally:

Click New Branch.
Name the branch descriptively, like feature/setup-inventory-playbooks.
Click Create branch to save it.

(screenshot)

click on new terminal on your vscode it will already be inside the directory
now run 
git pull origin main
(screenshot)

heck out the new branch locally:
bash
Copy code
git checkout -b feature/setup-inventory-playbooks
This creates and switches to the new branch on your local machine.
(screenshot)

Within your repository folder, create the following directories:
bash
Copy code
mkdir playbooks
mkdir inventory

(screenshot)

Create the common.yml Playbook:

Navigate to the playbooks folder:
bash
Copy code
cd playbooks
Create an empty playbook file called common.yml:
bash
Copy code
touch common.yml or instead of this just create new file under playbooks in vscode and name it common.yml

(screenshot)
This file will eventually hold tasks that will be run on the hosts.

now inside your VSCODE, create 4 new files namely dev, staging, uat, and prod
(screenshot)

Set Up Ansible Inventory (Defining Hosts)
Edit the Development Inventory File (dev):
Open the dev file in your vs code
Add the following inventory structure (replace the placeholder IPs with your actual private IP addresses):
ini
Copy code
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user=ec2-user

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user=ec2-user
<Web-Server2-Private-IP-Address> ansible_ssh_user=ec2-user

[db]
<Database-Private-IP-Address> ansible_ssh_user=ec2-user 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user=ubuntu

(screenshot)
then save the file

Set Up SSH Agent and Import Private Key
Ansible uses SSH to connect to the servers defined in the inventory. To allow Jenkins-Ansible to connect to these servers, you’ll need to load your private key into an SSH agent.

Start the SSH Agent:

Run this command to start the SSH agent:
bash
Copy code
eval `ssh-agent -s`

(screenshot)

search for your private key following the below

Navigate to the .ssh Directory

Run the following command to go to the .ssh directory:
bash
Copy code
cd ~/.ssh
List Files in the .ssh Directory

Once in the .ssh directory, list the files to see if your private key is there:
bash
Copy code
ls
Common names for private keys are id_rsa, id_dsa, or sometimes custom names. Private keys don’t have a file extension, while public keys will end with .pub (e.g., id_rsa.pub is the public key).
Identify Your Private Key File

Look for a file without an extension (like id_rsa).

(screenshot)

Copy code
ssh-add ~/.ssh/id_rsa
Note: If your key has a different name or is located in a different directory, use that path instead.
3. Verify the Key is Added
To confirm that your key was added successfully, use:
bash
Copy code
ssh-add -l
If successful, you should see the key's fingerprint, confirming that it’s ready for use.

(screenshot)

You can also try it this way as I encountered issues using the id_dsa and it kept saying permission denied(public key) so I used the location of my pem key file
eval ssh-agent -s
Agent pid 599

ssh-add "C:\Users\PrinceTee\Downloads\lampproject.pem"

ssh-add -l

(screenshot)

test the connection

ssh ubuntu@<your-public-ip>
(Replace <your-public-ip> with the public IP of your Jenkins-Ansible instance).

 Push Changes to GitHub
After setting up your directory structure, playbook, and inventory files, commit and push these changes to GitHub.

Add Files to Git:

bash
Copy code
git add .
Commit Your Changes:

Write a meaningful commit message:
bash
Copy code
git commit -m "Set up Ansible directory structure and development inventory"
Push the Changes to GitHub:

Push to your feature branch:
bash
Copy code
git push origin feature/setup-inventory-playbooks

 Connect VS Code to Your Jenkins-Ansible EC2 Instance
Using Visual Studio Code Remote SSH Extension allows you to connect and edit files directly on your Jenkins-Ansible server.

Open Visual Studio Code and install the Remote - SSH extension from the Extensions marketplace.
(screenshot)

In VS Code, press F1, type "Remote-SSH: Connect to Host," and press Enter.
Add a new SSH connection with:
plaintext
Copy code
ssh ubuntu@<your-public-ip>
(Replace <your-public-ip> with the public IP of your Jenkins-Ansible instance).
Once connected, you can edit files directly on the server as needed.

(sceenshot)



It is time to start giving Ansible the instructions on what you need to be performed on all servers listed in inventory/dev.

In common.yml playbook you created in vscode you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your playbooks/common.yml file with following code:

---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  become: yes
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest
   

- name: update LB server
  hosts: lb
  become: yes
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

(screenshot)

we also have an additional tasks which is below

Create a directory and a file inside it
Change timezone on all servers
Run some shell script

Here’s how to create a directory, change the timezone, and run a shell script:

yaml
Copy code
- name: additional tasks for all servers
  hosts: all
  become: yes
  tasks:
    - name: Create a directory
      file:
        path: /tmp/mydirectory
        state: directory

    - name: Create a file in the new directory
      file:
        path: /tmp/mydirectory/myfile.txt
        state: touch

    - name: Change timezone to UTC
      command: timedatectl set-timezone UTC

    - name: Run a shell script
      shell: /path/to/your/script.sh

Update GitHub with Your Code Changes
Check Status of Git Changes:

Run git status to see the modified files in your repository.
Add Files to Git:

Add only the specific files you’ve modified or created (such as common.yml).
bash
Copy code
git add playbooks/common.yml
Commit the Changes with a Descriptive Message:

bash
Copy code
git commit -m "Add common.yml playbook for server updates and configurations"
Push the Branch to GitHub:

If you’re on a feature branch, push your branch to GitHub.
bash
Copy code
git push origin feature/setup-inventory-playbooks

(screenshot)

Step 3: Create a Pull Request (PR) on GitHub
Go to Your GitHub Repository:

Navigate to your repository page on GitHub, and you should see a prompt to create a Pull Request for your recent push.
Create a Pull Request:

Click on “Compare & pull request.”
Add a descriptive title and comment about the changes.
Review the changes to ensure they’re accurate, then click Create pull request.
Review the Code as a Different Developer:

Imagine you’re a team member reviewing someone else’s code.
Check for any issues, and leave a comment if necessary.
Merge the Pull Request:

(screenshot)

If everything looks good, click Merge pull request and confirm.
Pull the Latest Changes to Your Local Master Branch:

Switch to the master branch.
bash
Copy code
git checkout master
Pull the latest changes from GitHub.
bash
Copy code
git pull origin master

(screenshot)

Step 4: Verify Jenkins Automation (if integrated with Jenkins)
Jenkins Build Trigger:

If you’ve set up Jenkins to trigger a job on code changes, Jenkins will automatically start a build once your code is merged into the master branch.

(screenshot)

Confirm Build Artifacts on Jenkins:

Log into Jenkins and navigate to your project.
Once the job runs, confirm the build output is saved in the /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory.

(screenshot)

Connect to Your Ansible-Controlled Server from VSCode
Open VSCode:

Start VSCode on your local machine.
Connect to Your Remote Instance:

In VSCode, use the Remote - SSH extension to connect to your server instance.
Press F1, type Remote-SSH: Connect to Host..., and select the server you wish to connect which in our case is 51.21.47.126. after that you will get prompted to choose from either linux windows or macos. choose linux

if you encounter an error pop up  saying "could not establish connection to "51.21.47.126 ": permission denied(publickey). " like I did you will need to update your ssh config file with th ssh key we used to set up the ssh remote connection. in my case I addedthe line IdentityFile "C:\Users\Prince-Tee\Downloads\lampproject.pem"
find below the screenshot

(Screenshot)

After adding that line Press F1, type Remote-SSH: Connect to Host..., and select the server you wish to connect which in our case is 51.21.47.126. I saw that ssh was connecting. find below the screenshot from the output in vscode

(screenshot)
(screenshot)

I checked terminal and so that the connection was succesfully

(sreenshot)

Prepare the Ansible Environment
Navigate to Your Ansible Directory:

Once connected to the server through VSCode, open the terminal in VSCode or on the remote server.
Navigate to your Ansible configuration directory where your playbook and inventory files are located:
bash
Copy code
cd ~/ansible-config-mgt
Confirm Inventory and Playbook Locations:
(screenshot)

Check that the inventory/dev.ini and playbooks/common.yml files are present in this directory:
bash
Copy code
ls inventory/dev.yml playbooks/common.yml
(screesnhot)

if they are not present run git pull origin main
(screenshot)

now run 

ls inventory/dev.ini playbooks/common.yml

then after that run 

Run the playbook with the following command:
bash
Copy code
ansible-playbook -i inventory/dev.ini playbooks/common.yml


if you encounter permission issue,
first go to your aws and make sure you Open the Inbound Rules of Each Server's Security Group:

Go to the security group associated with each server (DB, LB, web servers, and NFS).
Under the Inbound rules section, add a new rule for SSH.
Specify the Ansible Server’s Private IP:

For the Source field, set it to the private IP of your Ansible server, not a wide range like 0.0.0.0/0. This restricts SSH access to only the Ansible server, enhancing security. 