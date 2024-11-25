# AWS-Cybersecurity-Homelab

<h1>ABSTRACT</h1>

<p>In this project I’m creating a cloud-based cybersecurity home lab using Amazon Web Services (AWS). The purpose of this lab is to practice engineering a cloud-based network that can later be used as a lab environment for simulating red and blue team activities. Leveraging AWS, I provisioned three EC2 instances using Kali Linux as the attacker, Windows 11 as the target workstation, and a security box hosting Splunk SIEM and Nessus vulnerability scanner in Ubuntu. Each machine is configured for remote access using RDP for Windows, XRDP for Kali, and VNC for the Ubuntu security box. These three virtual machines are connected within an isolated network via a Virtual Private Cloud (VPC) with one public subnet, a route table, an internet gateway, and security groups defining what traffic is allowed for each virtual machine. Splunk is installed on the security box and configured to receive data from the Windows 11 machine using Splunk Universal Forwarder. Tenable Nessus is also installed on the security box to scan for vulnerabilities on the target machine.</p>



<h1>TOOLS AND ENVIRONMENTS</h1>
    <ul>
        <li>Amazon Web Services (AWS): EC2, VPC</li>
        <li>Operating Systems: Windows 10, Kali Linux, Ubuntu</li>
        <li>Splunk</li>
        <li>Nessus</li>
    </ul>



    
<h1>STEPS</h1>
    <ol>
        <li>Sign up for Amazon Web Services</li>
        <li>Install Windows Subsystem for Linux (WSL)</li>
        <li>Create a Virtual Private Cloud (VPC)
            <ul>
                <li>Public Subnet</li>
                <li>Route Table</li>
                <li>Internet Gateway</li>
            </ul>
        </li>
        <li>Create security groups
            <ul>
                <li>Homelab group</li>
                <li>Security group</li>
            </ul>
        </li>
        <li>Create a Key Pair</li>
        <li>Provision EC2 instances
            <ul>
                <li>Kali Linux</li>
                <li>Windows 11</li>
                <li>Ubuntu Desktop</li>
            </ul>
        </li>
        <li>Access Kali machine
            <ul>
                <li>Configure XRDP for Kali</li>
            </ul>
        </li>
        <li>Access Ubuntu Desktop
            <ul>
                <li>Change default VNC password</li>
            </ul>
        </li>
        <li>Access Windows machine through RDP
            <ul>
                <li>Turn off Windows Defender Firewall</li>
            </ul>
        </li>
        <li>Install Splunk to Ubuntu security box</li>
        <li>Install Splunk Universal Forwarder to Windows 11</li>
        <li>Create an Index in Splunk Enterprise</li>
        <li>Install Tenable Nessus</li>
    </ol>

<h1>Walk Through</h1>
    <ol>
      <li>To set up the lab environment, I signed up for Amazon Web Services and used the free tier products provided by AWS for the entire lab setup. Once the account is created, navigate to the AWS Console Homepage.</li>
      <li>I used a Windows computer to build this lab and installed Windows Subsystem for Linux (WSL) for full access to an Ubuntu terminal. WSL is used after provisioning the Kali machine to configure XRDP. I downloaded Ubuntu 24.04 from the Microsoft Store. Initially, the application threw an error, but there was a simple fix: go to Control Panel > Programs > Programs and Features > Turn Windows features on or off, then select Windows Subsystem for Linux to enable it.</li>
      <li>From the AWS Console, I located the VPC dashboard. The VPC is an isolated portion of the AWS cloud that will house the public subnet and its associated EC2 instances. In this section, I created a new VPC, assigned an IPv4 address range of 10.0.0.0/16 (providing 65,536 addresses), and added one public subnet. Once created, the associated subnet, route table, and internet gateway can be viewed under their respective tabs in the VPC dropdown menu.</li>       
      <li>The next step is to create the security groups. Security groups are listed in the VPC dashboard under the Security tab. A security group acts as a virtual firewall to control the traffic to EC2 instances. Once the instances are created, they will be added to a specified group. Two security groups are created with the following rules assigned to the VPC created in the previous step:

<h3>Cloud Homelab SG</h3>

  <h4>Inbound Rules:</h4>
  <ul>
    <li>SSH (Port 22) – Source: My Public IP</li>
    <li>RDP (Port 3389) – Source: My Public IP</li>
    <li>All ICMP IPv4 – Source: 0.0.0.0/0 (Any IP Address)</li>
  </ul>

  <h4>Outbound Rules:</h4>
  <ul>
    <li>All Traffic – Source: 0.0.0.0/0 (Any IP Address)</li>
  </ul>


<h3>Netspectrum's Ubuntu Desktop (Security Box)</h3>
  
  <h4>Inbound Rules:</h4>
  <ul>
    <li>All ICMP IPv4 – Source: 0.0.0.0/0 (Any IP Address)</li>
      <li>HTTPS (Port 443) – Source: 0.0.0.0/0 (Any IP Address)</li>
      <li>HTTP (Port 80) – Source: 0.0.0.0/0 (Any IP Address)</li>
      <li>RDP (Port 3389) – Source: 0.0.0.0/0 (Any IP Address)</li>
      <li>SSH (Port 22) – Source: 0.0.0.0/0 (Any IP Address)</li>
      <li>Custom UDP (Port 3389) – Source: 0.0.0.0/0 (Any IP Address)</li>
      <li>Custom TCP (Port 9997) – Source: 0.0.0.0/0 (Any IP Address)</li>
      <li>Custom TCP (Ports 5900-5920) – Source: 0.0.0.0/0 (Any IP Address)</li>
  </ul>

  <h4>Outbound Rules:</h4>
  <ul>
    <li>All Traffic – Source: 0.0.0.0/0 (Any IP Address)</li>
  </ul>

  The Cloud Homelab SG allows inbound SSH and RDP traffic from my public IP address and ICMP traffic from any IP address. It also allows all outbound traffic. The Security Box is configured with default rules from the AMI, allowing inbound traffic from HTTPS, HTTP, SSH, RDP, UDP port 3389, and TCP ports 5900–5920. Additional inbound rules allow all ICMP traffic from any source and TCP port 9997 for use by the Splunk Universal Forwarder on the Windows 10 virtual machine. All outbound traffic is allowed. </li>
      <li>Before provisioning the EC2 instances, a key pair is required for secure connection to the machines. The key pair consists of a public and a private key, with the public key used for encryption and the private key used for decryption. For this project, the three instances will use the same key pair. I selected RSA (Rivest-Shamir-Adleman) encryption for the key pair in the .pem file format. RSA is a reliable public-key cryptography algorithm that creates a public key from two large prime numbers, which can encrypt data, but only the holder of the private key can decrypt it. Once the .pem file is downloaded, I moved it from the downloads folder to the WSL Ubuntu folder in the file explorer.</li>
      <li>With the key pair ready, I can now create the EC2 instances. Amazon EC2 allows the creation of virtual machines using an Amazon Machine Image (AMI) template that contains the necessary software to launch the instance. This lab environment uses three AMIs: Kali Linux, Netspectrum's Ubuntu Desktop, and Windows Server 2025. Each instance is added to the Cybersecurity Homelab VPC and created using the key pair from the previous step. A public IP address is automatically assigned, and each instance is manually assigned to a security group. Kali and Windows are placed in the Homelab SG, while Ubuntu is assigned to the Netspectrum security group. Each instance is also assigned an instance type that determines its CPU capacity, memory, and storage. Kali uses t2.micro, Ubuntu uses t3.large, and Windows uses t3.micro. The three instances are now ready for use after basic configuration.</li>
      <li>To remotely log in to the Kali machine, it must be configured for RDP with Xfce. This can be done using WSL. Navigate to the Kali instance in the EC2 dashboard, select Connect, and then choose SSH client. Use the default username and public DNS or copy and paste the provided command into the WSL shell to access Kali. The following script will update Kali, install Xfce and XRDP, and configure XRDP to listen on port 3389 (the default RDP port):
<pre><code>#!/bin/sh
echo "[i] Updating and upgrading Kali (this will take a while)"
apt-get update
apt-get full-upgrade -y

echo "[i] Installing Xfce4 & xrdp (this will take a while as well)"
apt-get install -y kali-desktop-xfce xorg xrdp

echo "[i] Configuring xrdp to listen to port 3389 (but not starting the service)"
sed -i 's/port=3389/port=3389/g' /etc/xrdp/xrdp.ini
    </code></pre>
Change the password:
    <pre><code>echo kali:kali | sudo chpasswd</code></pre>

Start xrdp:
    <pre><code>sudo systemctl enable xrdp –now</code></pre>
Now that XRDP is configured and running, you can access the Kali GUI using Remote Desktop Connection on Windows. Using the public IPv4 address, connect to the machine and log in. The Kali setup is now complete</li>
    <li>The security box can be accessed using VNC (Virtual Network Computing). Using the public IP address associated with this instance in the EC2 dashboard, you can access the machine by entering the IP address in a web browser. A password prompt will appear, and the default password is the EC2 instance ID. Once connected, use the default prompt to change the VNC password. The security box is now ready for use, and it will be configured with Splunk and Nessus later.</li>
      <li>The final instance is Windows 11. To log in, a password is required, which is created using the key pair made earlier. In the EC2 dashboard, navigate to the instance, select Connect, and then choose RDP Client. In this section, select Get Password and upload the private key file. Use the .pem file that was created earlier to decrypt the password, which will be displayed in plaintext. Open Remote Desktop Protocol on your desktop and use the public IP address to connect. Use the default Administrator user and the new password to log in. To make the target machine more vulnerable to attacks and enable data transmission to Splunk, disable Windows Defender Firewall from the control panel under both private and public network settings. All three virtual machines are now ready for use.</li>
      <li>On the Ubuntu security box, open Firefox and sign up for Splunk Enterprise. Download the version for Linux. In the terminal, change directories to the Downloads folder, depackage the file, and start Splunk. Create a user in the terminal, and when prompted, open the Splunk web interface link. Log in using your credentials. Once logged in, go to Settings > Forwarding and Receiving > Configure Receiving. Create a new receiving port and configure Splunk to listen on port 9997. This port will be used by Windows to send data to Splunk.</li>
      <li>Now that Splunk is set up to receive data on the security box, configure Windows to send data. Log in to the Windows machine, download the Splunk Universal Forwarder for Linux, and run the installer. Choose Customize Options and select application, security, and event logs. Set a username and password, then enter the IP address of the Ubuntu security box and port 9997 in the receiving indexer page. Complete the installation.</li>
      <li>12.	To create a data repository (index) in Splunk, go to Settings > Indexes > New Index on the security box and create a new index called windows-security. To send Windows events to the new index, navigate to C:\Program Files\SplunkUniversalForwarder\etc\system\local, copy the outputs.conf file, and rename the copied file as inputs.conf. Open inputs.conf and add the following:
    <pre><code>
    [WinEventLog://Security]
    index = windows-security 
    disabled = 0</code></pre>
Save and close the file. Restart Splunk on Windows, and return to Splunk Enterprise on Ubuntu. A search for ‘index="windows-security"’ should now return events from the Windows machine. Splunk Enterprise and Splunk Universal Forwarder are now fully operational. </li>
    <li>The final step is to download and configure Nessus on the security machine. In Firefox, go to the Tenable Downloads page, select the latest version for Linux – Debian – amd64, and install it. Once installed, depackage the file, then start Nessus. Enter the IP address provided in the terminal into Firefox and register for Nessus Essentials. After the plugins finish compiling, Nessus will be ready to run vulnerability scans.</li>
    </ol>

<h1>Conclusion</h1>
<p>In this project, I successfully built a cloud-based cybersecurity home lab using Amazon Web Services (AWS) to simulate real-world network scenarios. The setup involved provisioning and configuring EC2 instances running Kali Linux, Windows 11, and Ubuntu, with each instance performing a distinct role in the simulated network environment. By integrating tools such as Splunk SIEM for security information and event management, and Tenable Nessus for vulnerability scanning, the lab provides a controlled environment for conducting red and blue team exercises.
The process of setting up the lab was an excellent hands-on experience in configuring cloud infrastructure, managing security groups, and deploying essential cybersecurity tools. Through this project, I gained a deeper understanding of network security operations and how to effectively implement monitoring, vulnerability management, and remote administration within a cloud-based environment. This lab setup is now a valuable resource for testing and developing cybersecurity skills and can be easily scaled or modified for future security exercises.
</p>

<h1>Lessons Learned</h1>
<h4>Cloud Infrastructure and Networking</h4>
<p>One of the key lessons learned was the importance of properly configuring AWS networking components, such as the Virtual Private Cloud (VPC), EC2 instances, subnets,  and security groups. Understanding how to isolate resources within a VPC and manage network traffic through security groups was essential for building a secure environment. I realized that even small misconfigurations in these components could lead to security vulnerabilities or connectivity issues.</p>
<h4>Remote Access Configuration</h4>
<p>Setting up remote access protocols like RDP for Windows, XRDP for Kali, and VNC for the Ubuntu security box presented a valuable learning opportunity. It was crucial to ensure that each machine was securely configured for remote access to avoid exposing vulnerabilities. Configuring XRDP on Kali Linux, in particular, was a bit tricky at first, but I learned that using the right terminal commands and configuring the services correctly can make remote access seamless.</p>
<h4>Problem Solving and Troubleshooting</h4>
<p>Throughout the setup, there were challenges such as dealing with initial setup errors (e.g., WSL configuration issues or remote access not working), which required careful troubleshooting. I learned to be patient and methodical in identifying problems whether it was network configuration, software installations, or service configurations. Each issue that arose deepened my understanding of the tools and services involved.</p>
<h4>Automation and Scalability</h4>
<p>While this lab was manually configured, I realized the potential for automating certain processes using AWS CloudFormation, Terraform, or other scripting methods. Automating the provisioning of EC2 instances, security groups, and VPC configurations would not only streamline the process but also make the lab environment more scalable. This is something I plan to explore in future projects to save time and reduce the chance for human error.</p>

<h1>Future Work</h1>
Although the initial goal of building a cloud-based cybersecurity home lab has been achieved, there are several improvements and enhancements I plan to implement in the future. First, I aim to harden the security of the current environment. Since this lab was designed as a sandbox for red and blue team exercises, it was not built with full security in mind, leaving some vulnerabilities exposed. For example, the security groups I created allow traffic from any IP address on several ports, which is not a security best practice. To address this, I will tighten the security of the network and services, ensuring they adhere to better security standards.
Additionally, I plan to redesign the network topology and possibly create a new VPC dedicated to the security tools box, further isolating sensitive resources.
Beyond security improvements, I intend to simulate the following attack scenarios within this lab environment:
<ul>
    <li>Insider Threat Simulation: Simulate an insider threat scenario where a legitimate user abuses their privileges to exfiltrate data or escalate their access. I will attempt to detect, respond to, and resolve the incident using the Ubuntu Security Box.
    </li>
    <li>Vulnerability Scanning and Remediation: Use Nessus to perform a comprehensive vulnerability scan on both the Kali and Windows machines. After identifying vulnerabilities, I will patch the systems and run another scan to confirm successful remediation. I will also monitor the patched vulnerabilities using Splunk and simulate attacks post-patching to verify the effectiveness of the fixes and collect data in Splunk.
    </li>
    <li>Phishing Attack Simulation: Launch a phishing attack using Kali Linux to target the Windows 10 machine via an interactive email, such as a fake login page or a malicious link. Once access is gained, I will simulate lateral movement within the network and exfiltrate data. Using Splunk, I will monitor the attack at each stage of the Cyber Kill Chain, from reconnaissance through to exfiltration, to refine detection and response strategies.   
    </li>
</ul>
By focusing on these areas, I can further enhance the lab’s functionality and security, providing valuable experience in simulating and defending against a variety of real-world cyber threats.
    

<h1>Sources</h1>


    
