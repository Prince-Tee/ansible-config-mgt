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



