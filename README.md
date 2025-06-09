Here is a complete step-by-step guide with all commands and details to set up your Jenkins-Ansible-Java deployment system using a JAR file:


---

🔧 Environment Assumptions

Jenkins is installed on one Ubuntu system.

Another Ubuntu system is used as the deployment target (10.0.2.15 in your case).

The Jenkins build produces a JAR file using Maven.

You want to copy the JAR and run it using ansible-playbook.



---

✅ 1. Install Ansible on Target Machine (10.0.2.15)

sudo apt update
sudo apt install ansible -y


---

✅ 2. Create Required Directories

On the target machine (10.0.2.15):

mkdir -p /home/ubuntu/deployment
mkdir -p /home/ubuntu/ansible-deploy


---

✅ 3. Create Ansible Inventory File

nano /home/ubuntu/ansible-deploy/hosts

Add this content:

[local]
localhost ansible_connection=local


---

✅ 4. Create Ansible Playbook

nano /home/ubuntu/ansible-deploy/deploy.yml

Paste the following content:

---
- name: Deploy built JAR file
  hosts: localhost
  become: yes
  tasks:

    - name: Stop existing Java process (if any)
      shell: pkill -f 'java -jar'
      ignore_errors: yes

    - name: Wait for 2 seconds after killing process
      pause:
        seconds: 2

    - name: Start the new JAR file in background
      shell: nohup java -jar /home/ubuntu/deployment/ansible1-1.0-SNAPSHOT.jar > /home/ubuntu/app.log 2>&1 &
      args:
        executable: /bin/bash


---

✅ 5. Verify Java is Installed

Run:

java -version

If not installed, install OpenJDK:

sudo apt install openjdk-17-jdk -y


---

✅ 6. Setup Passwordless SSH for Jenkins User

On Jenkins machine:

Switch to Jenkins user:

sudo su - jenkins

Generate SSH key:

ssh-keygen -t rsa

Copy public key to deployment server:

ssh-copy-id ubuntu@10.0.2.15

Test:

ssh ubuntu@10.0.2.15

It should log in without a password.


---

✅ 7. Jenkins Post-Build Shell Command

In your Jenkins project → Configure → Build → Execute Shell, paste:

scp target/ansible1-1.0-SNAPSHOT.jar ubuntu@10.0.2.15:/home/ubuntu/deployment/
ssh ubuntu@10.0.2.15 "ansible-playbook -i /home/ubuntu/ansible-deploy/hosts /home/ubuntu/ansible-deploy/deploy.yml"


---

✅ 8. Run Jenkins Job

It will:

1. Compile with Maven.


2. Copy the JAR to the target machine.


3. Trigger Ansible to stop any old process and run the new one.


4. Log output to /home/ubuntu/app.log.





---

✅ 9. Monitor Application Logs (on target)

tail -f /home/ubuntu/app.log


---

Let me know if you want to:

Run this on a remote group of servers

Deploy as a systemd service instead of using nohup

Use Jenkins Pipeline instead of freestyle project


