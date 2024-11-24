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


    
