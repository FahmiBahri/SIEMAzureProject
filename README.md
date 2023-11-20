Here is a detailed walkthrough of the steps I took to create a SIEM system. This project revolved around Microsoft Azure and therefore, my first step was to create a Microsoft Azure account. 

**Step 1 - Create an Azure account**
Whilst creating this project, Azure were running a promotion allowing new users to obtain $200 credit upon signup. This will help cover the costs of running this project.
![Free Azure Account](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/4db3a1ed-f249-4a63-b919-252580198b80)

**Step 2 - Create a virtual Machine**
The virtual machine (VM) isn't required yet but it does take a long time to provision. Therefore, I decided to create it now and let it provision in the background whilst focussing on other tasks.
![Create VM](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/045f092e-0b29-4473-bab9-28c23209b8ad)

This VM is going to be the 'device' that I make available for external threat actors to attack. In essence, it is going to be my 'honeypot'. This VM is a resource and therefore needs to be a part of a Resource Group. So I created a Resource Group called 'HoneyPotLab' to collect all resources I plan to use during this project. 
![Create VM - Inbound Ports](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/283d3c14-92dd-4209-8fb7-1e4934efd392)

For now, I decided to limit my inbound traffic to port 80 as there was no option to allow all incoming traffic. My options were: HTTP (80), HTTPS (443) or SSH (22). Later on, I will change this to allow traffic from all IP addresses.

**Step 2b - Create a Network Security Group**
No disk requirements are necessary outside of the default settings for this project so I left that section on default and proceeded straight to the 'Networking' stage. In this section, I changed the 'NIC network security group' option to 'Advanced' and clicked the 'Create new' button to create a new Network Security Group (NSG). I also checked the box next to 'Delete public IP and NIC when VM is deleted' to ensure that the NSG is deleted along with this VM when I am finished with the project.
![Create VM - Advanced NSG](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/8801c33d-03aa-4280-94ca-75f0632e62de)

When in this section, I deleted the current inbound rule and clicked '+ Add an inbound rule'.
![NSG - Delete inbound rules](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/d2ffa318-9227-4887-9415-466be56fd2fb)

To gain a lot of data, I altered the settings of my new rule to allow traffic to come from any port and be received through any destination port, using any protocol. In usual circumstances, this would NOT be a good idea. However, I've done this on this project to attract as many attackers as possible so I had to make the VM highly discoverable.
![NSG - New inbound rule](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/6eeefb95-b833-4d40-b9f3-8f985f42c0b8)

No other settings need to be amended, so I then clicked 'Review + create'. Then finally, I clicked 'Create' to finalise the creation of the VM.
![Create VM - Final step](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/ec01417a-9caf-4f30-b5a0-cec0431bd04e)

**Step 3 - Create a Log Analytics Workspace**
The next step is to create a Log Analytics Workspace. I used the search bar at the top of Azure to find this resource.
![Create LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/d3f30770-7d18-4a5e-a935-91100cc3ba25)

As I already have created a resource group dedicated to this project, I allocated this resource there. Next, I clicked on 'Review + Create' and then 'Create'. Eventually, the deployment was complete.
![LAW deployment complete](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/7e128f48-37b8-4a5a-8b75-846b41703a68)

Next, I went on to the Security Center to access Microsoft Defender for Cloud. This is to allow the logs from the VM to be aggregated in the LAW. In the Security Center, I clicked on the 'Microsoft Defender for Cloud' hyperlink near the top of the page.
![Security Center](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/03b6996c-a0bb-481a-a440-d05f9a7878e9)


In Microsoft Defender for Cloud, I clicked on 'Environment settings' and then the drop-down button next to my subscription. This revealed my LAW, which I then clicked on. Here, I can connect services like Microsoft Defender and Microsoft Sentinel to my VM to help collect date for my LAW.
![MDfC](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/b2851fc5-0108-44cc-9f5c-4d6ffcf2a1de)

This opened up 'Defender plans' and they were all 'Off' by default. For this project, I will need to enable the 'Foundational CSPM' and 'Servers' plans. I do not plan on using SQL servers so did not need to activate that plan. Once I done this, I clicked 'Save'. After it saved successfully, I clicked on the 'Data collection' option.
![CSPM and Servers plans](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/2daa298c-ac16-4a37-ac59-e45a4c36f5d5)

The next step was to enable the option for 'All Events' to be collected - by default it is set to 'None'.
![Data Collection](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/c7205379-ad63-46d6-8fb5-50dbaac63a8f)


After this, I connected my LAW to the VM. To do this I went back into my LAW (LAW-HoneyPot), clicked 'Virtual Machines (deprecated)' and clicked on the VM I made earlier (HoneyPot-VM).
![VM connection to LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/bc289f69-4c2e-4124-b51c-82a16196045f)
![VM connection to LAW 2](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/1fc058b4-1da9-4be5-9bde-1ce695044506)


**Step 4 - Microsoft Sentinel connection**
The next step was to create and connect Microsoft Sentinel to my VM.
![Microsoft Sentinel connection](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/27bc7f02-44f8-47a2-85d1-6463527db40b)
![Add Microsoft Sentinel to LAW](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/301cd774-c224-48c7-9dfc-a3e69a6bafc5)

At this stage, the required configurations have now beem completed to begin setting up the VM itself. To set up a remote desktop connection (RDP), I first needed the IP address of the VM. I found this in the 'Virtual Machines' section of Azure. 
![VM Public IP Address](https://github.com/FahmiBahri/SIEMAzureProject/assets/151456646/66d51116-85bc-4edc-9910-47c7be699500)

**Step 5 - Remote Desktop connection to VM**

