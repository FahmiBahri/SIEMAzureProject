<h2>Overview</h2>


Here is a detailed walkthrough of the steps I took to create a SIEM system. This SIEM system displays threat actor data on a world map, using Microsoft Sentinel.



In this project, I create a virtual machine on Azure and turn off most security settings to make it vulnerable to potential threat actors. Once my virtual machine gets recognised by threat actors, I extract data from security logs of their failed attempts and plot this data on a world map to show where these attacks are coming from. The data extraction involves using a third-party API. At the end of the project, I am left with a map containing location data of threat actors.



<h2>Toolkit</h2>

- Microsoft Azure
- Microsoft Sentinel
- Azure Log Analytics Workspace
- Virtual Machines
- Remote Desktop Connection
- PowerShell
- IPGeolocation API
- Windows Event Viewer
- Windows Firewall



<h2>Environments</h2>

- Microsoft Azure
- Windows 10



<h2>Walkthrough</h2>

This project revolved around Microsoft Azure and therefore, my first step was to create a Microsoft Azure account. 

<h4>Step 1 - Create an Azure account</h4>

Whilst creating this project, Azure were running a promotion allowing new users to obtain $200 credit upon signup. This will help cover the costs of running this project.

![Free Azure Account](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/4db3a1ed-f249-4a63-b919-252580198b80)



<h4>Step 2 - Create a virtual Machine</h4>



![Create VM](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/045f092e-0b29-4473-bab9-28c23209b8ad)



The virtual machine (VM) isn't required yet but it does take a long time to provision. Therefore, I decided to create it now and let it provision in the background whilst focussing on other tasks.



![Create VM - New Resource Group](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/7f4b9809-6774-4435-9b04-07966c3a0001)



This VM is going to be the 'device' that I make available for external threat actors to attack. In essence, it is going to be my 'honeypot'. This VM is a resource and therefore needs to be a part of a Resource Group. So I created a Resource Group called 'HoneyPotLab' to collect all resources I plan to use during this project. 



Resource Groups in Azure are very useful as they allow you to create logical connections between tools, services and other resources. This is particularly useful for allocating resources to different projects. Due to the logical grouping of these resources, anything stored in a resource group can be provisioned or deleted together. This feature makes resource management a smooth process in Azure.



![Create VM - Inbound Ports](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/3d0ebaec-2f21-4e37-9b31-f0af6d88454e)



I will be connecting to this VM later using an RDP connection. Therefore, in the 'Inbound Port Rules' section, I made sure to allow 'RDP (3389)' traffic inbound.



<h4>Step 2b - Create a Network Security Group</h4>



![Create VM - Create VN](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/14b8561b-d636-4334-a872-3a172a80e874)



No disk requirements are necessary outside of the default settings for this project so I left that section on default and proceeded straight to the 'Networking' stage. 



The first step here was to create a virtual network.



![Create VM - Create Public IP](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/3644f76a-2da3-435f-aabe-6a05303e16aa)



Then I assigned a public IP to the VM.



![Create VM - Advanced NSG](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/8801c33d-03aa-4280-94ca-75f0632e62de)



In this section, I changed the 'NIC network security group' option to 'Advanced' and clicked the 'Create new' button to create a new Network Security Group (NSG). I also checked the box next to 'Delete public IP and NIC when VM is deleted' to ensure that the NSG is deleted along with this VM when I am finished with the project.



An NSG is fundamentally a firewall for Azure resources. It uses rules to grant or deny access to and from a particular Azure resource, covering both inbound and outbound traffic.



![NSG - Delete inbound rules](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/d2ffa318-9227-4887-9415-466be56fd2fb)



When in this section, I deleted the current inbound rule and clicked '+ Add an inbound rule'.



![NSG - New inbound rule](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/6eeefb95-b833-4d40-b9f3-8f985f42c0b8)



To gain a lot of data, I altered the settings of my new rule to allow traffic to come from any port and be received through any destination port, using any protocol. In usual circumstances, this would NOT be a good idea. However, the aim of this project is to attract as many attackers as possible. Therefore, making the VM highly discoverable was paramount.



![Create VM - Final step](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/9d1d97ce-2b22-4e03-b670-36afe1fe4a8b)



No other settings need to be amended, so I then clicked 'Review + create'. Then finally, I clicked 'Create' to finalise the creation of the VM.




<h4>Step 3 - Create a Log Analytics Workspace</h4>



![Create LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/cfbf90ec-2a2f-4a85-ad3b-5d0eaa5967b8)



The next step is to create a Log Analytics Workspace (LAW). This will be used to collect sign-in attempt logs from the VM. I used the search bar at the top of Azure to find this resource.



![LAW deployment complete](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/7e128f48-37b8-4a5a-8b75-846b41703a68)



As I already have created a resource group dedicated to this project, I allocated this resource there. Next, I clicked on 'Review + Create' and then 'Create'. Eventually, the deployment was complete.



![Security Center](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/03b6996c-a0bb-481a-a440-d05f9a7878e9)



Next, I went on to the Security Center to access Microsoft Defender for Cloud. This is to allow the logs from the VM to be aggregated in the LAW. In the Security Center, I clicked on the 'Microsoft Defender for Cloud' hyperlink near the top of the page.



![MDfC](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/b2851fc5-0108-44cc-9f5c-4d6ffcf2a1de)



In Microsoft Defender for Cloud, I clicked on 'Environment settings' and then the drop-down button next to my subscription. This revealed my LAW, which I then clicked on. Here, I can connect services like Microsoft Defender and Microsoft Sentinel to my VM to help collect date for my LAW.



![CSPM and Servers plans](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/2daa298c-ac16-4a37-ac59-e45a4c36f5d5)



This opened up 'Defender plans' and they were all 'Off' by default. For this project, I will need to enable the 'Foundational CSPM' and 'Servers' plans. I do not plan on using SQL servers so did not need to activate that plan. Once I done this, I clicked 'Save'. After it saved successfully, I clicked on the 'Data collection' option.



![Data Collection](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/c7205379-ad63-46d6-8fb5-50dbaac63a8f)



The next step was to enable the option for 'All Events' to be collected - by default it is set to 'None'.


<h4>Step 4 - Connecting the LAW to the VM</h4>

![VM connection to LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/bc289f69-4c2e-4124-b51c-82a16196045f)



To do this I went back into my LAW (LAW-HoneyPot), clicked 'Virtual Machines (deprecated)' and clicked on the VM I made earlier (HoneyPot-VM).



![VM connection to LAW 2](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/1fc058b4-1da9-4be5-9bde-1ce695044506)



Then I clicked the 'Connect' button.



<h4>Step 5 - Microsoft Sentinel connection</h4>



![Microsoft Sentinel connection](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/27bc7f02-44f8-47a2-85d1-6463527db40b)



The next step was to create and connect Microsoft Sentinel to my VM. With Sentinel open, I clicked 'Create Microsoft Sentinel'.



![Add Microsoft Sentinel to LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/301cd774-c224-48c7-9dfc-a3e69a6bafc5)



Then I clicked on my VM, and pressed the 'Add' button.



![Sentinel - Add LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/c1687b92-c748-4357-aad2-53cf2804a787)



Finally, I added the Sentinel to my LAW.



<h4>Step 5 - Remote Desktop connection to VM</h4>



![VM Public IP Address](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/66d51116-85bc-4edc-9910-47c7be699500)



At this stage, the required configurations have now beem completed to begin connecting to the VM itself. I am connecting via a remote desktop connection (RDP), which will require the public IP address of the VM. I found this in the 'Virtual Machines' section of Azure.



![RDP - VM Connection](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/ab1b0338-6158-4e23-97e1-03b008e30104)



Then I opened the 'Remote Desktop Connection' app on my desktop and inputted the VM's public IP address and clicked 'Connect'.



![RDP - Certificate Error](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/0909c9a5-a8c9-42b5-b6eb-f4cb3e5318d0)



After this I logged on successfully and accepted the identity certificate warning dialogue box.



![RDP - VM Monitor](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/4a4cffc2-615c-451d-bf41-7586b013cbb6)



![VM RDP - Network Discovery](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/a27e6dcf-94b8-4445-9df1-52bd2e975d79)



Then the monitor loaded. The first thing I done was enable 'network discovery' so that my VM is more discoverable to threat actors.



![VM - Event Viewer](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/d7a6c351-5478-46a6-a4d3-d1f5c32a1a44)



The next step was to open 'Event Viewer' on my VM.



![VM - Security Logs](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/14b40218-f097-4a3c-ab02-c9cd54900e05)



Once loaded, I clicked on 'Windows Logs' and then 'Security'.



![VM - Failed Logs](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/405fad1e-6ca4-4d48-a97c-e4b8fc5342db)



![VM - Failed Log Report](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/67fe9b95-70a2-4d93-9579-e4718e6df5f8)




Here is an example report of a failed login attempt. It contains valuable information, like the source IP address.

My next step was to take abstracts of this information to make it more useful. I decided that I would utilise a program that will scan the source network IP address of attempted logins and reveal their geographical location. To do this, I needed to make use of an API - which I sourced from: ipgeolocation.io.



![Ping Timeout VM](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/2533abc4-4248-4b3c-af32-ce853864f365)



Before doing this, I decided to turn off the VM's firewall to make it even easier to discover. I know that it is not easily penetrable right now because when I try to ping it from my actual PC, my ping request times out.



![VM - Firewall On](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/6064f531-f51e-4795-a395-8bbd28133577)



I went to my firewall settings on the VM. All firewall settings were currently on.



![VM - Firewall Off](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/0ec5dd67-9ead-4adf-8f44-f93ef73c3b8e)



![Ping Success VM](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/57fb2007-6cc5-4d5d-8aa5-cd3d588eb340)



I disabled all firewalls. Now I'm able to ping my VM from my PC.



<h4>Step 6 - Collecting Security Log Data from the VM</h4>



![VM - Log Exporter Save](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/fc898fef-9302-4768-b8c7-0773261fd5bb)



The next step was for me to utilise a PowerShell script to export my VM's security logs onto a document. Whilst you can create a custom script, I made use of one that was already available online. 
I pasted this script into PowerShell ISE on the VM and saved the document.



![IP Geo Location - API Key](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/482e55ec-00b3-4064-bfba-b3d567855aed)



![VM - API Key Update](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/75f8eabc-7e87-4ee9-982a-23e15eeb27b9)



I then updated the API key with my personal one so that it works correctly for my resources and saved it again.



![VM - Run PS Script](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/00216aa4-bffd-4045-9202-5e2b9ec8c3c0)



After saving the document, I run the script. This script has been written in a way that it perpetually loops. Therefore, it will always extract the security logs from my VM and take geographical data from the IP addresses of failed login attempts. This will then be exported onto the text document file that has been generated. 



![VM - Failed Logs Document](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/80aae54d-7092-45d9-af42-08439c52bd0c)



The script I used exports the data onto a document in the 'ProgramData' folder of the VM. However, this folder was hidden so I needed to enable view of hidden documents.



![VM - Example Log](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/888d5748-64c0-4e27-8e11-32567c1d7b0c)



Here is an example entry of a failed login attempt that was recorded in the file.



<h4>Step 7 - Exporting the Log Data onto Azure</h4>



![LAW - New Custom Log](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/e4992fd3-d0c3-48e9-827a-d17373e7b6b7)



To do this, I need to create a custom log on my LAW that will bring in the data from that document saved on my VM's 'ProgramData' folder.

I went onto my LAW and clicked on: 'Tables', '+ Create' and then 'New custom log (MMA-based)'.



![VM - Failed Logs Document](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/fd42cea9-1f79-45ec-a5ba-37d32f97faef)



The document containing the failed logs was saved onto my VM. So I just copied the contents of that document and saved it onto a notepad document on my PC.



![LAW - Open Sample Log](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/d4d8e24c-7854-4d63-84cf-c61640f42955)



![LAW - Custom Log Record Delimiter](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/1a71f375-1092-466b-9724-4d96576ed391)



Then I clicked 'Next'. For the 'Record delimiter', I left it on 'New line' as my file contents were logs where each entry was separated by a line.



![LAW - VM Collection Path](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/80aa8084-cda2-4436-bd67-fff644b7b2ca)



Next, I had to define the collection path for the document containing the logs. To do this, I went onto the folder containing the document in File Explorer on the VM. I then clicked on the path at the top of the window and copied it. I also took note of the file name.



![LAW - Custom Log Collection Path](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/404e2c81-d77d-4316-83ba-dd87fa6c2f19)



For this step, I first chose 'Windows' as the 'Type'. Then I pasted the path I copied earlier into the 'Path' text box, followed by a backslash and then the file name. It's important to put the file extension at the end, otherwise an error message appears. So for this, I used '.log' as the file extension. Then I clicked 'Next'.



![LAW - Custom Log Name](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/b2551b5c-467e-47e4-9665-335b9d2bfa6d)



Then I input a suitable name for the custom log.



![LAW - Custom Log Create](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/585d056f-aef6-4df8-a4ee-c6b8031b83e9)



Finally, I reviewed the details and created the custom log. It can take a while for the VM and the LAW to synchronise properly. 



![LAW - Custom Log Not Ready](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/16200fc2-347f-4a29-8122-e4681b533b2e)



As a result, I couldn't query that table straight away with the data I was looking for - which is failed RDP login attempts.



![LAW - Security Logs](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/75b09a9f-4366-4cd7-a075-3631cf810e9a)



However, a few moments later, I attempted the query again. This time, I received some data.



![VM - Failed Login Event ID](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/8aa5a4f7-caf1-4e05-8fb5-20d55a9acb25)



![LAW - EventID Query](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/3c7ce9dc-d3b2-4a49-8a15-bf63ee5d39d9)



Another way to find failed login attempts was to query them using the EventID. In the VM's security logs, failed login attempts have an EventID which is: 4625. On my LAW, I queried that EventID to get the same data.



![LAW - Raw Data on Failed RDP Logs](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/21a292e5-42a5-45d4-915b-e23fb60198ed)



![LAW - Raw Data Extraction](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/6a746389-6369-40cd-9948-35f4a9ca0358)



Going back to my original 'Failed_RDP_With_Geo_CL' query, I needed to now extract location details from the raw data from the results. In my LAW, I began a new query. In the query, I ran this script to extract the information I needed from the raw data:



Failed_RDP_With_Geo_CL 
| extend username = extract(@"username:([^,]+)", 1, RawData),
         timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
         latitude = extract(@"latitude:([^,]+)", 1, RawData),
         longitude = extract(@"longitude:([^,]+)", 1, RawData),
         sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
         state = extract(@"state:([^,]+)", 1, RawData),
         label = extract(@"label:([^,]+)", 1, RawData),
         destination = extract(@"destinationhost:([^,]+)", 1, RawData),
         country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost"
| where sourcehost != ""
| summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country



<h4>Step 8 - Creating a visual map</h4>

The next step I took was to create a visual map that shows the geographical location of the host IP addresses attempting to log into my VM.



![Sentinel - Add Workbook](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/e4c9db66-6ffc-420f-9ff2-4636779a4655)



On Sentinel, I clicked on 'Workbooks' and then '+ Add Workbook'



![Workbook - Edit Workbook](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/3848bcee-7c1e-4958-aab5-662d613f4e28)



![Workbook - Remove Default Queries](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/0072476b-3e57-4263-b736-d759cda6426f)



Next, I clicked 'Edit' and removed the two basic analytics queries that come as default when creating the workbook. 



![Workbook - Add Query](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/325a6ce3-31c2-40d8-adb5-1d6d77393603)



The next step was to run a new query. In this query, I did not want to include any of the sample logs as I wanted to only show the actual login attempts from threat actors. To do this, I first needed to start a new query. So I clicked the '+ Add' drop down menu, then 'Add query'.



![Workbook - Correct Query](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/ad940559-dad4-4840-9e87-2912c692ad64)



Then, I added the query I used earlier for the raw data extraction in LAW:

Failed_RDP_With_Geo_CL 
| extend username = extract(@"username:([^,]+)", 1, RawData),
         timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
         latitude = extract(@"latitude:([^,]+)", 1, RawData),
         longitude = extract(@"longitude:([^,]+)", 1, RawData),
         sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
         state = extract(@"state:([^,]+)", 1, RawData),
         label = extract(@"label:([^,]+)", 1, RawData),
         destination = extract(@"destinationhost:([^,]+)", 1, RawData),
         country = extract(@"country:([^,]+)", 1, RawData)
| where destination != "samplehost"
| where sourcehost != ""
| summarize event_count=count() by latitude, longitude, sourcehost, label, destination, country

Then I clicked 'Run Query' and my data appeared.



![Workbook - Map Visualisation](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/fb738e17-d29e-4d0e-b8be-392a132163d5)



The next step was to change the 'Visualization' to 'Map'.



![Workbook - Map Layout Settings](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/6d8d646c-2e29-4c84-85e1-2b81e744af09)



In 'Map Settings', I changed the 'Location Info using' option to 'Latitude/Longitude'.



![Workbook - Map Metric Settings](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/5d0e0740-195a-4f22-87fb-8f9387755a81)



Before saving and closing my Map Settings, I had to edit my metrics. I changed the 'Metric Label' option to 'label' and 'Metric Value' to 'event_count'. I left the rest of the options on their default settings. Then I clicked 'Apply' and finally 'Save and Close'.



![Workbook - Save Workbook](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/6750f8d2-1a9f-407d-9c5a-0aa9797f01e7)



![Workbook - Failed RDP World Map](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/49d341ed-1e11-4db9-a1f9-720b2dc612a4)



For now, I'm done with the map and this workbook so I saved and closed it.



![World Map Refresh](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/c5d58b89-e6f8-4adb-a16a-42641e0baea5)



![World Map Refreshed](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/ccf25bbd-e626-4648-a650-28d95dba8500)



![World Map Refreshed 2](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/fd7a7f9f-7324-48ef-a6d4-5f79659d1905)



To make sure it is working properly, I refreshed it to see if the log counts would change. Thankfully, they did.


