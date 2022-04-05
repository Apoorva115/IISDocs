# Title 

 Configure Alerts via Log Analytics to notify the admins whenever the configuration store of the IIS websites are modified. 

# Scenario 


Consider you are an admin of the server hosting business critical applications. To minimize the production impact incase of any issues, you would like to configure alerts so that whenever the website is stopped or started or any configuration changes are made to the config store, you will get notified and you can take corrective measures. 

# Pre-requisties 

  - Please make sure to have Azure VM created already. If not, please follow the article for step-by-step instructions: : https://docs.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-portal.
  - Make sure to have IIS feature already installed on the server. If not, refer the article for step-by-step instructions: https://docs.microsoft.com/en-us/iis/application-frameworks/scenario-build-an-aspnet-website-on-iis/configuring-step-1-install-iis-and-asp-net-modules


# Implementation 

For the purpose of this demo, we will configure the alerts whenever the website is started or stopped. Please note that, for this feature implementation, we will be using Configuration auditing. 

IIS configuration auditing is a feature that would let you monitor the changes that are done to the IIS configuration store. It generates event messages and display the configuration element which was changed, user who initiated the change, and the original and the new value of the element

To enable Configuration auditing, follow the below steps: 

	1. Open Event viewer from the server
	2. Go to Application and Services Logs -> Microsoft -> Windows -> IIS Configuration -> Operational. 
	3. Right click on Operation and "Enable Log"

By Default  the logs would be saved in 
**%SystemRoot%\System32\Winevt\Logs\Microsoft-IIS-Configuration%4Operational.evtx.** You can change it by clicking on the “Properties”, and changing the “Log Path”

After enabling, you would see corresponding events in this event viewer log whenever there is a change to the IIS configuration store.

Since we are focusing on website start and stop events, below is an example of how event generated looks like when default website is started: 

![image](https://user-images.githubusercontent.com/81897348/161762368-c06f47a2-ee65-49b9-a9e0-efd1c293219c.png).

If you expand and check the details section, we see below: 

![image](https://user-images.githubusercontent.com/81897348/161762531-60e3d048-01be-46dc-b61e-4306c0528faf.png).


This event also shows the username who initiated the change. This is a very important data if you are tracking who changed certain IIS configuration. This in fact gives you the Security Identifier of the username (SID), and you can find the username by using psgetsid which is a part of our Windows Sysinternals tools 
You can also see what were the changes made, details about the configuration where the change was made, the old value of the configuration element, and the new value.

Since we need to fire alerts when website is started or stopped, we will pick up the "started" and "stopped" keyword that we see in the event data and use it to add under the alerts section. 


Now that the server configuration part is done, lets move on to Azure Portal to configure alerts. 
We will make use of Loganalytics workspace to configure the alerts. 

1. Open the Azure portal and go to Log Analytics Workspace. 
2. We need to connect the VM to the Log analytics workspace. To do that, follow the below steps: 
 - In In Log analytics workspace -> Virtual machines -> select VM -> Connect. 
 - Once connected successfully to this workspace, we will see as below:
	![image](https://user-images.githubusercontent.com/81897348/161764246-2a28acc4-74bc-41c4-91df-ae7f30cc355f.png).
3. Open the Default workspace and select Agents configuration
4. Click on "Add windows events logs" and select "Microsoft-IIS-Configuration/Operational" from the dropdown. 

![image](https://user-images.githubusercontent.com/81897348/161708690-6172b8e9-2ae6-4800-ba12-72d3d216e8aa.png)
5. Now to validate if the workspace is connected properly to the VM, we can go to "Logs" under the General section and run below two queries: 
- Run "heartbeat" and make sure we see connectivity happening with the VM
- Run "events" query and make sure we see event activity shown below.

![image](https://user-images.githubusercontent.com/81897348/161717653-6509ac45-c654-4e70-88e6-8aca01f66d64.png)
6. Once we have confirmed that connectivity is succesfully established, then we can add the below events for website start and stop accordingly:



	Event
	| where EventLog == "Microsoft-IIS-Configuration/Operational"
	| where EventData contains "STARTED"
	
	
	Event
	| where EventLog == "Microsoft-IIS-Configuration/Operational"
	| where EventData contains "STOPPED"

![image](https://user-images.githubusercontent.com/81897348/161718102-2b9a37f6-5800-4002-a99a-066e3109a41d.png)

	

	1. Then we will configure the alert by clicking on " + New alert rule".
	2. The Condition tab will include the log query which is pasted above and we are configuring the alert to be fired every 15 minutes and check the data for every last 10 minutes.  Condition tab is where you can define what actions and notifications are triggered when the alert rule generates an alert
	3. Under the Actions Tab, we will create EmailAlert by selecting the "Select Action groups". Specify the email address where you want the alert to be received. 
	4. Under the Details Tab, specify the "Alert rule details" like  severity of the rule etc.
	5. Under Advanced options, we select "enable upon creation"

![image](https://user-images.githubusercontent.com/81897348/161718323-ee01e8de-0a06-4bd6-a8cb-a307d50b1130.png)
	
	6. You can specify tags if required. 
	7. Then you can review and create the rule. 


You can refer more on how to manage/configure alerts from the article: https://docs.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-activity-log



We have now succesfully configured log analytics to trigger alert whenever website is started/stopped from the on-prem IIS server. 

# Testing

1. Once rule is configured, we can test if the rule is created succesfully by either starting or stopping the website on Azure VM
2. Once started, you will receive an email notification from Microsoft Azure with subject as below for example: "Fired:Sev3 Azure Monitor Alert IISWebsiteStatus on ftpfileshare ( microsoft.compute/virtualmachines ) at 4/1/2022 12:22:43 PM" 


	![image](https://user-images.githubusercontent.com/81897348/161765557-df58bb6d-a832-47f2-941b-70655a1df985.png)



# Conclusion 
	
By following the above mentioned steps, we have succesfully configured alerts from Loganalytics which will fire the alert whenever the website on IIS server is started or stopped.
