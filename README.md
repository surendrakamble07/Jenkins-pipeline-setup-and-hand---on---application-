# Jenkins-pipeline-setup-and-hand-on-application-

✅ JENKINS MASTER–AGENT SETUP (SIMPLE & CORRECT)

We have:

Master EC2 → Jenkins installed

Agent EC2 → Only Java + SSH

User on agent → ubuntu

🔹 STEP 1: Generate SSH key on MASTER (as Jenkins user)

On MASTER EC2:

sudo su - jenkins


Create SSH key:

ssh-keygen -t ed25519 -f ~/.ssh/jenkins_agent -N ""


This creates:

/var/lib/jenkins/.ssh/jenkins_agent
/var/lib/jenkins/.ssh/jenkins_agent.pub

🔹 STEP 2: Copy PUBLIC key to AGENT

On MASTER:

cat ~/.ssh/jenkins_agent.pub


Copy the full line starting with:

ssh-ed25519 AAAA...

🔹 STEP 3: Add public key to AGENT (ubuntu user)

Login to AGENT EC2:

ssh ubuntu@<AGENT_PUBLIC_IP>


Run:

mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys


➡ Paste the public key
➡ Save & exit

Set permission:

chmod 600 ~/.ssh/authorized_keys

🔹 STEP 4: Test SSH from MASTER (MOST IMPORTANT)

On MASTER:

sudo su - jenkins
ssh -i ~/.ssh/jenkins_agent ubuntu@<AGENT_PUBLIC_IP>

✅ Expected:

No password

Direct login

If this works → Jenkins WILL work.

🔹 STEP 5: Add SSH key to Jenkins Credentials

Jenkins UI:

Manage Jenkins → Credentials → (Global) → Add Credentials


Fill:

Kind: SSH Username with private key

Username: ubuntu

Private Key: Enter directly
👉 Paste content of:

cat /var/lib/jenkins/.ssh/jenkins_agent


ID: surendra

Save.

🔹 STEP 6: Configure Jenkins Agent (Node)
Manage Jenkins → Nodes → New Node


Fill:

Node name: surendra

Remote root directory:

/home/ubuntu/jenkins


Labels: surendra

Launch method: Launch agents via SSH

Host: AGENT_PUBLIC_IP

Credentials: ubuntu (surendra)

Host key verification: Non-verifying

Save.

🔹 STEP 7: Launch Agent 🚀

Click:

Launch agent


✅ Node status → ONLINE

🔹 STEP 8: Run Pipeline on Agent
pipeline {
    agent { label 'surendra' }

    stages {
        stage('Test') {
            steps {
                sh 'whoami'
                sh 'hostname'
                sh 'echo "Hello from Jenkins Agent"'
            }
        }
    }
}

❌ WHAT MISTAKES YOU MADE (IMPORTANT)
❌ Mistake 1: Generated SSH key as root
/root/.ssh/id_ed25519


🔴 Jenkins does NOT use root
✔ Jenkins runs as jenkins user

❌ Mistake 2: Added public key to WRONG user on agent

You added key in:

/root/.ssh/authorized_keys


✔ It must be:

/home/ubuntu/.ssh/authorized_keys

❌ Mistake 3: Used different SSH keys

You used:

Windows .pem

Root key

Jenkins key (mixed)

✔ Jenkins must use exact same key that worked in manual SSH

❌ Mistake 4: Jenkins credential had WRONG private key

Even though:

ssh -i jenkins_agent ubuntu@agent


worked,

Jenkins credential still had a different key, so SSH failed.

🧠 GOLDEN RULE (REMEMBER FOREVER)

Manual SSH must work using the same key, user, and host before Jenkins will work.
