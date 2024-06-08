<h1>Creating an Azure Sentinel Map with Live Attacks | Beginner SIEM Honeypot Lab</h1>
<p>In this lab, we will set up Azure Sentinel (SIEM) and connect it to a Windows 10 virtual machine. This virtual machine will be set up as a honeypot with minimal security to increase its vulnerability. We will then observe live attacks (RDP Brute Force) from all around the world. Data to the Azure Sentinel map will be plotted using geolocation information retrieved from the VM through Log Analytics workspace through a custom PowerShell script written by Josh Madakor, the creator of this lab activity.</p>

<b>INSERT IMAGE</b> 
<br>
To begin the lab, we need to sign up for an Azure account. At the time of writing this, Azure provides $200 of free credits to practice their services (https://azure.microsoft.com/).
<h3>Creating Resource Group for the Lab</h3>
Once an account is created, you will be introduced to the home page. We will create a Resource Group container to hold all the resources needed in this lab. Next, find the Resource Group icon under Azure Services or use the search bar at the top.
<br>
<b>INSERT IMAGE</b> 
<br>
Once you are on the Resource Group page, create a group with a relevant name to this lab and select the default region or the closest region to you.
<br>
<b>INSERT IMAGE</b> 
<br>
<h3>Creating a Honeypot VM</h3>
After creating the Resource Group, we will navigate to the Virtual Machine page. Here we will create a Windows 10 VM that will be used as our honeypot for the lab.
