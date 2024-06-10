<h1>Creating an Azure Sentinel Map with Live Attacks | Beginner SIEM Honeypot Lab</h1>
<p>In this lab, we will set up Azure Sentinel (SIEM) and connect it to a Windows 10 virtual machine. This virtual machine will be set up as a honeypot with minimal security to increase its vulnerability. We will then observe live attacks (RDP Brute Force) from all around the world. Data to the Azure Sentinel map will be plotted using geolocation information retrieved from the VM through Log Analytics workspace through a custom PowerShell script written by <a href="https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1">Josh Madakor</a>, the creator of this lab activity.</p>
<p align="center">
  <picture>
    <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/1%20lt4BL9WOV4FyVzpwO2YB_g.webp/ alt="Lab Diagram" width="70%" height="70%"">
  </picture>
  <br>
  <em>Diagram showing the layout of the tools used in this lab.</em>
</p>
<br>
To begin the lab, we must sign up for an Azure account. At the time of writing this, Azure provides $200 of free credits to practice their services (https://azure.microsoft.com/).
<h3>Creating Resource Group for the Lab</h3>
Once an account is created, you will be introduced to the home page. We will create a Resource Group container to hold all the resources needed in this lab. Next, find the Resource Group icon under Azure Services or use the search bar at the top.
<br>
<p align="center">
  <picture>
    <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/1%20w4iWP195k6UJPF8ruQYwcw.webp alt="Azure Home Page"">
  </picture>
</p>
Once on the Resource Group page, create a group with a relevant name to this lab and select the default region or the closest region to you.
<br>
<p align="center">
  <picture>
    <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/1%205QGmkxao1mhvl4AOMZKCAw.webp alt="Create Resource Group" width="70%" height="70%""> 
  </picture>
</p>
<br>
<h3>Creating a Honeypot VM</h3>
After creating the Resource Group, we will navigate to the Virtual Machine page. Here we will create a Windows 10 VM that will be used as our honeypot for the lab.
<br><br>
 <ol>
  <li>Select the Honeypot Resource Group and choose a Windows 10 image with minimal size under Instance details.</li>
  <li>Set up an admin account with credentials you will remember.</li>
   <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/CreateVM.webp alt="Create Virtual Machine" width="70%" height="70%""> 
    </picture>
   </p>
  <li>Select Next: Disks and leave options as default.</li>
  <li>Select Next: Networking and change NIC Network Security Group to advanced and select Create network security group.</li>
    <p align="center">
      <picture>
        <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/VMNetworkConfig.webp alt="Virtual Machine Network Settings" width="70%" height="70%""> 
      </picture>
   </p>
  <li>Create a new inbound rule for this network security group to allow all traffic as seen below. This will make the VM easier to discover and more vulnerable to attacks.</li>
   <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/NSG.webp alt="Network Security Group Settings" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Once the network security group is set up, create the VM after reviewing all settings.</li> 
</ol> 
<h3>Setting up Log Analytics Workspace</h3>
<p>Allow 5–20 minutes for the VM to complete setting up. During this time we will set up a Log Analytics workspace. This will be used to ingest logs collected from the VM. We will then use these logs to create custom logs containing geographic information. Log Analytics workspace will store these custom logs for Azure Sentinel where we will create a map to discover the location of the attacks.
</p>
<p>To begin, navigate to the Log Analytics workspace page and create a new workspace within the honeypot Resource Group as seen below:
</p>
<p align="center">
      <picture>
        <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/Create%20LAW.webp alt="Log Analytics Workspace" width="70%" height="70%""> 
      </picture>
</p>
<p>Now that we have a Log Analytics workspace created, we will need to enable the ability to gather logs from the VM into the workspace. Follow the steps below to set up log gathering:</p>
<ol>
  <li>Navigate to the Windows Defender for Cloud page and select the Log Analytics Workspace previously created:</li>
  <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/MicrosoftDefenderLAW.webp alt="Defender LAW selection" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Enable the Microsoft Defender plan for servers only as we are not working with any SQL servers in this lab.</li>
</ol>
  
