# Title 

 Configure Alerts via Log Analytics to notify the admins whenever the configuration store of the IIS websites are modified. 

# Scenario 


Consider you are an admin of the server hosting business critical applications. To minimize the production impact incase of any issues, you would like to configure alerts so that whenever the website is stopped or started or any configuration changes are made to the config store, you will get notified and you can take corrective measures. 

# Pre-requisties 

  - Please make sure to have Azure VM created already. If not, please follow the article: : https://docs.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-portal.
  - Make sure to have IIS feature already installed on the server. If not, refer the article: https://docs.microsoft.com/en-us/iis/application-frameworks/scenario-build-an-aspnet-website-on-iis/configuring-step-1-install-iis-and-asp-net-modules


# Implementation 

For the purpose of this demo, we will configure the alerts whenever the website is started or stopped. Please note that, for this feature implementation, we will be using Configuration auditing. 

IIS configuration auditing is a feature that would let you monitor the changes that are done to the IIS configuration store. It generates event messages and display the configuration element which was changed, user who initiated the change, and the original and the new value of the element

To enable Configuration auditing, follow the below steps: 

	1. Open Event viewer from the server
	2. Go to Application and Services Logs -> Microsoft -> Windows -> IIS Configuration -> Operational. 
	3. Right click on Operation and "Enable Log"

By Default  the logs would be saved in 
%SystemRoot%\System32\Winevt\Logs\Microsoft-IIS-Configuration%4Operational.evtx. You can change it by clicking on the “Properties”, and changing the “Log Path”

After enabling, you would see corresponding events in this event viewer log whenever there is a change to the IIS configuration store.

Since we are focusing on website start and stop events, below is an example of how event generated looks like when default website is started: 

| Changes to '/system.applicationHost/sites/site[@name="Default Web Site" and @id="1"]/@state' at 'MACHINE/WEBROOT/APPHOST' have successfully been committed |

If you expand and check the details section, we see below: 

EventData
PhysicalPath
\\?\C:\WINDOWS\system32\inetsrv\config\applicationHost.config
ConfigPath
MACHINE/WEBROOT/APPHOST
EffectiveLocationPath
Configuration
/system.applicationHost/sites/site[@name="Default Web Site" and @id="1"]/@state
EditOperationType
1
OldValue
Starting
NewValue
Started

This event also shows the username who initiated the change. This is a very important data if you are tracking who changed certain IIS configuration. This in fact gives you the Security Identifier of the username (SID), and you can find the username by using psgetsid which is a part of our Windows Sysinternals tools (usage: psgetsid <SID>).
You can also see what were the changes made, details about the configuration where the change was made, the old value of the configuration element, and the new value.




