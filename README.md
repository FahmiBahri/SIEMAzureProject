Here is a detailed walkthrough of the steps I took to create a SIEM system. This project revolved around Microsoft Azure and therefore, my first step was to create a Microsoft Azure account. 

**Step 1 - Create an Azure account**
Whilst creating this project, Azure were running a promotion allowing new users to obtain $200 credit upon signup. This will help cover the costs of running this project.
![Free Azure Account](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/4db3a1ed-f249-4a63-b919-252580198b80)



**Step 2 - Create a virtual Machine**



![Create VM](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/045f092e-0b29-4473-bab9-28c23209b8ad)



The virtual machine (VM) isn't required yet but it does take a long time to provision. Therefore, I decided to create it now and let it provision in the background whilst focussing on other tasks.



![Create VM - New Resource Group](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/7f4b9809-6774-4435-9b04-07966c3a0001)



This VM is going to be the 'device' that I make available for external threat actors to attack. In essence, it is going to be my 'honeypot'. This VM is a resource and therefore needs to be a part of a Resource Group. So I created a Resource Group called 'HoneyPotLab' to collect all resources I plan to use during this project. 



![Create VM - Inbound Ports](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/3d0ebaec-2f21-4e37-9b31-f0af6d88454e)



I will be connecting to this VM later using an RDP connection. Therefore, in the 'Inbound Port Rules' section, I made sure to allow 'RDP (3389)' traffic inbound.



**Step 2b - Create a Network Security Group**



![Create VM - Create VN](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/14b8561b-d636-4334-a872-3a172a80e874)



No disk requirements are necessary outside of the default settings for this project so I left that section on default and proceeded straight to the 'Networking' stage. The first step here was to create a virtual network.



![Create VM - Create Public IP](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/3644f76a-2da3-435f-aabe-6a05303e16aa)



Then I assigned a public IP to the VM.



![Create VM - Advanced NSG](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/8801c33d-03aa-4280-94ca-75f0632e62de)



In this section, I changed the 'NIC network security group' option to 'Advanced' and clicked the 'Create new' button to create a new Network Security Group (NSG). I also checked the box next to 'Delete public IP and NIC when VM is deleted' to ensure that the NSG is deleted along with this VM when I am finished with the project.



![NSG - Delete inbound rules](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/d2ffa318-9227-4887-9415-466be56fd2fb)



When in this section, I deleted the current inbound rule and clicked '+ Add an inbound rule'.



![NSG - New inbound rule](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/6eeefb95-b833-4d40-b9f3-8f985f42c0b8)



To gain a lot of data, I altered the settings of my new rule to allow traffic to come from any port and be received through any destination port, using any protocol. In usual circumstances, this would NOT be a good idea. However, I've done this on this project to attract as many attackers as possible so I had to make the VM highly discoverable.



![Create VM - Final step](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/9d1d97ce-2b22-4e03-b670-36afe1fe4a8b)



No other settings need to be amended, so I then clicked 'Review + create'. Then finally, I clicked 'Create' to finalise the creation of the VM.




**Step 3 - Create a Log Analytics Workspace**



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


**Step 4 - Connecting the LAW to the VM**

![VM connection to LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/bc289f69-4c2e-4124-b51c-82a16196045f)



To do this I went back into my LAW (LAW-HoneyPot), clicked 'Virtual Machines (deprecated)' and clicked on the VM I made earlier (HoneyPot-VM).



![VM connection to LAW 2](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/1fc058b4-1da9-4be5-9bde-1ce695044506)



Then I clicked the 'Connect' button.



**Step 5 - Microsoft Sentinel connection**



![Microsoft Sentinel connection](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/27bc7f02-44f8-47a2-85d1-6463527db40b)



The next step was to create and connect Microsoft Sentinel to my VM. With Sentinel open, I clicked 'Create Microsoft Sentinel'.



![Add Microsoft Sentinel to LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/301cd774-c224-48c7-9dfc-a3e69a6bafc5)



Then I clicked on my VM, and pressed the 'Add' button.



![Sentinel - Add LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/c1687b92-c748-4357-aad2-53cf2804a787)



Finally, I added the Sentinel to my LAW.



**Step 5 - Remote Desktop connection to VM**



![VM Public IP Address](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/66d51116-85bc-4edc-9910-47c7be699500)



At this stage, the required configurations have now beem completed to begin connecting to the VM itself. I am connecting via a remote desktop connection (RDP), which will require the public IP address of the VM. I found this in the 'Virtual Machines' section of Azure.
