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
<p>Once an account is created, you will be introduced to the home page. We will create a Resource Group container to hold all the resources needed in this lab. Next, find the Resource Group icon under Azure Services or use the search bar at the top.</p>
<p align="center">
  <picture>
    <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/1%20w4iWP195k6UJPF8ruQYwcw.webp alt="Azure Home Page"">
  </picture>
</p>
<p>Once on the Resource Group page, create a group with a relevant name to this lab and select the default region or the closest region to you.</p>
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
    <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/EnableDefenderPlan.webp alt="Enable Defender Plan" width="80%" height="80%""> 
    </picture>
  </p>
  <li>After enabling the plan, select the Data Collection option on the left side of the page and set it to store All Events.</li>
  <li>Return to the Log Analytics workspace page in Azure and navigate to virtual machines under Workspace Data Sources.</li>
   <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/ConnectVMToLAW.webp alt="Connect VM to LAW" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Select the connect option at the top of the page to connect the VM to the workspace.</li>
</ol>

<h3>Setting Up Data Collection and Custom Log</h3>
<p>Once the virtual machine is connected to the workspace we will log into the VM to prepare the machine to be our honeypot. Follow the steps below to setup the VM.</p>
<ol>
  <li>Navigate to the Virtual Machines page in Azure and select the honeypot VM. Gather the public IP address of the machine.</li>
  <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/VMPublicIP.webp alt="Find VM Public IP Address" width="80%" height="80%""> 
    </picture>
  </p>
  <li>Launch the Remote Desktop Connection app on your location machine and connect to the public IP of your VM on port 3389 (ie. 52.170.41.131:3389)</li>
  <li>Use the admin credentials set during the creation of the VM to log in.</li>
</ol>
<p>We will now load a script used to gather Windows Event security logs to gather information on the locations the attacks originate from. The script will grab all failed RDP login attempts on the VM and gather information on the location of the attack using https://ipgeolocation.io/, a free IP geolocation API. The script will receive the geolocation info from the service and store it in a log for our Log Analytics workspace to gather. This is required because the security logs in Event Viewer only display the IP address used by the attacker, and not their geographical information. Note that these locations may not be the real location of the attackers as VPNs can be used to mask the original IP/location of the attacker. However, this will help us in learning the IP addresses used to perform the attacks and visualize the concentration of attacks through a heatmap. Follow the steps below to set up the script for this lab:</p>
<ol>
  <li>Download the script created by Josh Madakor here: https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1</li>
  <li>Copy the script to a convenient location on the honeypot VM.</li>
  <li>Create an account for https://ipgeolocation.io/ to receive your API key for the script.</li>
   <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/IPGeoAPIKey.webp alt="IPGeolocate API Key" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Open the script on the VM and paste in the API key. Also, note the file path of the logfile.</li>
   <p align="center">
    <picture>
      <img src=https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/AddKeyToScript.webp alt="Add API Key to Script" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Close the editor and right click the script file, select Run with Powershell.</li>
</ol>
<p>The failed_rdp.log file will now begin populating with all failed login attempts with the information provided by the geolocation API and the VM’s Event Viewer. We must now create a custom log in our Log Analytics workspace that can bring in the information from failed_rdp.log into our workspace. Follow the steps below to create the custom log:</p>
<ol>
  <li>Copy the failed_rdp.log file from the VM onto your local machine.</li>
   <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/14%20-%20Failed%20RDP%20Logins%20Log.webp" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Navigate to Log Analytics Workspace page in Azure, select Custom Logs, and Add custom Log.</li>
   <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/15%20-%20Add%20Custom%20Log%20to%20LAW.webp" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Provide the file copied to your local machine as the sample. Check the Record delimiter page to confirm that the logs are displaying properly.</li>
  <li>Select next to the Collection paths page and provide the file path to the failed_rdp.log file on the honeypot VM.</li>
   <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/16%20-%20Create%20Custom%20Log%20In%20LAW.webp" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Select next to the Details page and provide a name for the custom log. We use FAILED_RDP_WITH_GEO_CL for this lab.</li>
  <li>Confirm settings and create the custom log.</li><br>
   <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/17%20-%20Review%20and%20Create%20Custom%20Log.webp" width="75%" height="75%""> 
    </picture>
  </p>
</ol>
<p>After giving a few minutes for the custom log to load, you can now run queries in Log Analytics workspace to view the information from the log. Typing in the name of the custom log will return the data from the log on the VM. We will need to extract the information from the raw data log to use in creating our map in Azure Sentinel.</p>
<p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/18%20-%20Run%20Query%20On%20Failed_RDP%20logs.webp" width="75%" height="75%""> 
    </picture>
  </p>
<p>To extract the information from the raw data into custom fields, follow the steps below:</p>
<ol>
  <li>Select any of the lines shown from the results of querying FAILED_RDP_WITH_DEO_CL and click on the options and select “Extract Fields…” You will be brought to the custom fields page.</li>
  <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/19%20-%20Extract%20Fields%20from%20Failed_RDP.webp" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Highlight the first field of the rawdata, latitude, and provide it with a relevant title ie. LATITUDE_CF, set the Field Type accordingly. NOTE: Sample results will be shown which help with configuring the custom log.</li>
  <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/20%20-%20Extract%20Latitude%20Field.webp" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Check the results to ensure only latitude data is highlighted, if correct save extraction. Repeat these steps for each field.</li>
  <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/21%20-%20Confirm%20Field%20Values.webp" width="75%" height="75%""> 
    </picture>
  </p>
  <li>To fix incorrect results, select the icon in the top right of the result and correctly highlight the data for the field. If there are several incorrectly highlighted fields, this will help in correcting others as well.</li>
   <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/22%20-%20Fix%20Incorrect%20Matches.webp" width="75%" height="75%""> 
    </picture>
  </p>
  <li>Once fields are extracted for each data type, you will find each custom field under its tab as seen below:</li>
   <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/23%20-%20Custom%20Fields%20List.webp" width="75%" height="75%""> 
    </picture>
  </p><br>
</ol>
<p>If you run the query another time after performing these steps, you will see that the rawdata for logs after we created our custom fields are categorized to the fields that were set. These fields will be used to create our Azure Sentinel Map.</p>
 <p align="center">
    <picture>
      <img src="https://github.com/thossa000/Knowledge-Base/blob/main/Beginner%20SIEM%20Honeypot%20Lab/images/24%20-%20Run%20Query%20to%20View%20Custom%20Fields.webp" width="75%" height="75%""> 
    </picture>
  </p>
<p>To extract the information from the raw data into custom fields, follow the steps below:</p>
<ol>
  <li>Select any of the lines shown from the results of querying FAILED_RDP_WITH_DEO_CL and click on the options and select “Extract Fields…” You will be brought to the custom fields page.</li>
  <li>Highlight the first field of the rawdata, latitude, and provide it with a relevant title ie. LATITUDE_CF, set the Field Type accordingly. NOTE: Sample results will be shown which help with configuring the custom log.</li>
  <li>Check the results to ensure only latitude data is highlighted, if correct save extraction. Repeat these steps for each field.</li>
  <li>To fix incorrect results, select the icon in the top right of the result and correctly highlight the data for the field. If there are several incorrectly highlighted fields, this will help in correcting others as well.</li>
  <li>Once fields are extracted for each data type, you will find each custom field under its tab as seen below:</li>
</ol>
<p>If you run the query another time after performing these steps, you will see that the rawdata for logs after we created our custom fields are categorized to the fields that were set. These fields will be used to create our Azure Sentinel Map.</p>

<h3>Creating Azure Sentinel Heatmap of Login Attempts</h3>
<p>After our custom log has been created with its fields, we will set up a new workbook in Azure Sentinel that will be used as our heatmap for the data being collected. The heatmap is a great tool to visualize the volume of attacks received by our honeypot with data on the location and IPs that the attacks originate from. Follow the steps below to create the Sentinel workbook:</p>
<ol>
  <li>Navigate to the Azure Sentinel page in Azure, select Workbooks from the list of options under Threat Management.</li>
  <li>Select Add workbook, a new workbook will be created with a few default widgets, remove the widgets by clicking Edit at the top of the screen and selecting the remove option from the drop down of each widget.</li>
  <li>Once the default widgets are removed, select Add query from the Add option at the top of the page.</li>
  <li>Use the following query to retrieve the custom fields data of each log from our workspace that does not include incomplete/sample data:</li>

```
FAILED_RDP_WITH_GEO_CL
|summarize event_count=count() by SOURCEHOST_CF, LATITUDE_CF, LONGITUDE_CF, COUNTRY_CF, LABEL_CF, DESTINATIONHOST_CF

| where DESTINATIONHOST_CF != “samplehost”

| where SOURCEHOST_CF != “”

| where COUNTRY_CF != “”
```
<li>Set Visualization to Map and Size to Full to output the data to the desired format.</li>
<li>Configure the Map Settings to use Latitude/Longitude data to correctly pin the attacks and set the size and aggression as seen below.</li>
<li>Configure the Color and Metric settings to appropriately visualize the data by volume of attacks per location.</li>
<li>Apply the new settings, you will now see a visual change in the map to represent the attacks with data of the attackers country and IP addresses.</li>
<li>Change the name of the new workbook and save.</li>
</ol>
<p>We will now leave the VM running the script for a few hours to collect more attack logs. These can be viewed live from the VM as the Powershell window will display the log of each attack as it happens. The workbook can be viewed by navigating to Azure Sentinel and viewing “My Workbooks” in the workbook page.</p>


