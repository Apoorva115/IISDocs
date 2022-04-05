# Title
Configuring FTP Site on  Azure VM having home directory pointing to Azure Storage account with File share. 

# Objective
We will walk through on how FTP can be configured on an Azure VM. In addition to this, we will walk through how FTP's home directory can be configured pointing to the Azure storage account having File Share. 
		

# Scenario

Consider a scenario where you are the server admin. You have given the responsibility of creating FTP site on server. There is a custom requirement to store the FTP files on Azure Storage account having a Fileshare.

# Pre-requisites:

1. For this demo, make sure we already have Azure VM deployed. If you don’t have one already, please refer the article on steps for creating a VM on Azure: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-portal.
2. Once VM is ready, please install IIS and FTP server. Follow the link: 
	- IIS: https://docs.microsoft.com/en-us/iis/application-frameworks/scenario-build-an-aspnet-website-on-iis/configuring-step-1-install-iis-and-asp-net-modules
	- FTP: https://docs.microsoft.com/en-us/iis/publish/using-the-ftp-service/scenario-build-an-ftp-site-on-iis. 
3. We need to have Azure  storage account created and mapped to a file share.
	- Please refer below steps on creating storage account: https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal.
4. For the purpose of this demo, I have created Storage account with name "azurefileshareak". 
	- I am using Data storage as "File shares". 
	- Once storage account is created, go to "File shares" located under "Data Storage" 
		![image](https://user-images.githubusercontent.com/81897348/161428279-a2a5092d-6887-4f18-bbeb-1cc26ff8a613.png).

5. Create  a FileShare , for this demo, I have created Fileshare with name "azurefileshareak".  Inside this, I have folder created with name "FTPFileShare". 
6. Putting all these together, we need to make a note of below three points which we will later use while configuring the FTP server on Azure VM. 



| Type | Value  								           |
| ---- | ---- |
| Storage account Name | AzureFileshareak |
| File share URL       | https://azurefileshareak.file.core.windows.net/azurefileshareak                           |
| Access Keys          | Ogxu*---*HhGULsTJ***BqYlrZsMWy6LL/V*---MsN****5/oMa7wTM+a6DW***aZFEJAO+ASt+XQOAw==  |	
7. To test the functionality , we need to have Filezilla client installed on the VM. Here is the link to install Filezilla: https://filezilla-project.org/download.php?platform=win64
	
	
# Implementaion:

1. Before creating FTP site, we need to create user having same name as the storage account name and Password as the access keys. 
	- Open lusrmgr.msc on the IIS Azure VM
	- Right click -> Create new user under "Users"
	- Make sure to give username and password same as Storage account name and access keys respectively
	- Add the user under the IIS_IUSRS group. 

![image](https://user-images.githubusercontent.com/81897348/161435344-09eaf978-07bc-4c18-8106-545063d21cb4.png)

2. Open IIS on the server. 
3. Create FTP site using the below steps: 
   - Right click on the Site -> New FTP site
   - Give FTP site name and Under physical Path , provide the path of the File share. 			
	![image](https://user-images.githubusercontent.com/81897348/161435409-8746bd96-943a-4e73-9527-6cf717a6f4fa.png).	
   - We need to add under "Basic settings" ->  "Connect as" -> specify the storage account name and the access keys.



![image](https://user-images.githubusercontent.com/81897348/161435442-78938dbf-fb7f-4f71-8eac-4964eed27dd2.png).

4. Once this step is done, you have successfully configured the FTP site on Azure VM with File share on storage account. 
 
		
		
		
# Testing: 

1. To test if the configuration is working properly, lets connect with FTP client Filezilla to the Azure VM
1. Download Filezilla client for windows OS.
1. Open the Filezilla and enter the host as your VM IP address or localhost.
1. Based on your requirement , add the username - password with "anonymous" user or "Basic auth" user and click on "Quick connect". 
1. We will be succesfully able to see the fileshare folder "FTPFileShare" created on our storage account as below. 

![image](https://user-images.githubusercontent.com/81897348/161435758-bf241b63-1ca2-4c86-8b60-bf209a9b2800.png)

# Conclusion: 

By following the above steps, we have succesfully configured FTP site having home directory pointing to Azure File share on storage account. 
