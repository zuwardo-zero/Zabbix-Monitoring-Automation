# Zabbix-Monitoring-Automation

Tech Stack:Virtual Box, Host Ubuntu 24.04 (Server), VM Linux Mint 22 (Agent/Node), Zabbix 7.0 (LTS), MySQL, Nginx, Bash.

1. Project Overview :
   
This personal documentation covers the deployment of a centralized monitoring solution capable of real-time performance tracking and automated incident response (Self-Healing). I built a Zabbix 7.0 environment from scratch, integrated a remote Linux node, and configured script automation to restore failed web services without manual intervention.

3. Architecture & Implementation

Phase A: Envirnoment Setup
In Virutal box, we can start by creating the virtual machine in which we will set as the nginx server, Mint OS is used for it's lightweight packages.
<img width="1300" height="736" alt="Screenshot from 2026-01-02 21-16-56" src="https://github.com/user-attachments/assets/50836386-3322-4846-99e0-fadb92ecf86a" />

Additionally, I configured the network using a bridge adapter between my Ubuntu host and the Mint VM, a simple ping command can verify the connectivity and the IP addresses are as shown.
   <img width="1205" height="685" alt="Screenshot from 2026-01-02 21-22-46" src="https://github.com/user-attachments/assets/ad4d4dfc-3bc7-43ce-a7b7-463e5a1fadbd" />
Database Backend: Installed and secured a MySQL database, manually tuned the schema using utf8mb4 binary collation to support Zabbix 7.0's high-performance requirements.
<img width="1366" height="768" alt="Screenshot from 2025-12-31 22-59-56" src="https://github.com/user-attachments/assets/94924536-f3fa-4d02-9f23-08106ff3cd9c" />
Important note: 
For security best practises and in a production envirnoment, we should use stronger credentials to prevent any unauthorized access.

Next, we'll need to download the necessary zabbix packages in both machines.
<img width="1207" height="690" alt="Screenshot from 2026-01-02 21-30-32" src="https://github.com/user-attachments/assets/e1eabf35-bff0-42a3-aaae-0da9e2b10b18" />
<img width="1366" height="768" alt="Screenshot from 2026-01-01 00-25-00" src="https://github.com/user-attachments/assets/53eff934-2a29-45bb-b758-2df304f7e78b" />
In the agent machine we'll have to modify the configuration file in order to connect to the zabbix server.
<img width="1366" height="768" alt="Screenshot from 2026-01-01 00-29-30" src="https://github.com/user-attachments/assets/e1d339da-c611-42f7-bc81-bccfddb8eb3f" />
To install the nginx setup we can type the following command :

```sudo apt install nginx```

also we should create a firewall rule to allow traffic on port 10051 (Zabbix default port) for incoming agent data.

```sudo ufw allow 10500/tcp```

Phase B: Monitoring and troubleshooting

We should be able to find an  interface in localhost/zabbix in which we'll have to input the previous sql credentials to import our database schema and follow the installation process.
<img width="1203" height="550" alt="Screenshot from 2026-01-02 21-58-50" src="https://github.com/user-attachments/assets/21bc7d61-5591-4935-9d55-09d7c7f7638f" />
After finishing the installation, the default credentials to login should be admin:zabbix, which must be changed for security purposes, after using those credentials we can have access to the dashboard.
<img width="1202" height="623" alt="Screenshot from 2026-01-02 22-03-34" src="https://github.com/user-attachments/assets/ac4b9eed-d0d9-46cb-ba13-157423074fc5" />
The first issue we notice is low storage in my host machine, we can dismiss this out of scope problem since I gave uneccessary space to the VM mint.
Now we need to add the VM into the hosts by choosing Data collection --> Hosts and then adding the VM information such as the IP address as shown.
<img width="1148" height="623" alt="Screenshot from 2026-01-02 22-13-22" src="https://github.com/user-attachments/assets/6561340f-32a7-4e90-bb62-da394b271dfb" />
A green ZBX in the availabilty column shows that our connection setup is successful.
We can also execute commands from our linux CLI.
<img width="1148" height="623" alt="Screenshot from 2026-01-02 22-19-14" src="https://github.com/user-attachments/assets/33bbe1b4-2057-48fa-a2b3-6ec9f1a0b2ff" />
Another test we can do is to apply some stress on our host server in order to check if the data is colleceted.
<img width="1148" height="623" alt="Screenshot from 2026-01-02 22-22-49" src="https://github.com/user-attachments/assets/11deddfe-11da-4ede-9322-b80a8c405b7f" />

Phase C:Restoring failed web services with script automation
In this test we will configure an automated script in order to restore nginx back on after a failure.

First we will configure the paramaters for our vm, and add it to the nginx template.
<img width="1155" height="618" alt="Screenshot from 2026-01-02 22-46-52" src="https://github.com/user-attachments/assets/fb6817e6-92ec-4995-aba9-030432eb5e21" />
We can stop the nginx service for now by typing the following command in the machine agent.

```sudo systemctl stop nginx```

Let's also add the incident response command in the sudoers file of the agent machine to require no passwords, this should not cause a security problem since it does not involve critical privileges or access to any files.
<img width="1366" height="768" alt="Screenshot from 2026-01-01 01-50-20" src="https://github.com/user-attachments/assets/cfa28c70-7e16-4746-876e-d3c03c479e6c" />
Next we should create the automated respone script by heading to Alerts --> scripts and write the command
<img width="1155" height="618" alt="Screenshot from 2026-01-02 22-54-32" src="https://github.com/user-attachments/assets/3a7b173b-c9e2-478a-aef0-1944c102bf59" />
Keep in mind that we can also execute it manually from the server CLI with the following command 

```zabbix_get -s 192.168.1.147 -k "system.run[sudo systemctl start nginx]"```

We can create the operation by heading to Alerts --> Actions and add the script we made
<img width="1147" height="611" alt="Screenshot from 2026-01-02 23-11-08" src="https://github.com/user-attachments/assets/42b257f9-e984-4890-a548-23135778fc47" />

Finally, we can check that the command was successfuly executed by going to Reports --> Action logs
<img width="1366" height="768" alt="Screenshot from 2026-01-01 01-58-00" src="https://github.com/user-attachments/assets/79cb346b-72ee-46c1-878c-0b534666c18f" />

Summary:
With the correct configuration for zabbix server and agents, we can created automated scripts to troubleshoot and fix common service failure without the need for human intervention.

















