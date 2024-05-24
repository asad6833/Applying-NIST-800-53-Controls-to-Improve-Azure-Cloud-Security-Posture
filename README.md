# Applying NIST 800-53 Controls to Improve Azure Cloud Security Posture
![Screenshot 2023-09-07 193428](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/7c92f149-d82b-46a8-b688-7b5213de3727)

## Introduction
In my Azure SOC-Honeypot Lab, both the Windows VM and Linux VM were receiving high volumes of malicious activity, including password brute force attempts on Azure AD, Microsoft SQL Server, and Linux SSH Server, as well as DDoS attacks. These incidents were a result of improperly configured security settings, which had exposed the virtual machines to the public internet, permitting traffic from any IP address on any port.

As part of the lab project, I intentionally permitted traffic from any IP address to serve as an example of poor security settings. This was done to generate logs and gather data about malicious actors on the internet. Having now collected this valuable data, the next step involves appropriately configuring the security settings to ensure the safety of the cloud environment from potential attacks, based on the NIST 800-53 Security Controls framework.

## NIST 800-53 Security Controls Framework
The control from NIST 800-53 that deals with securing network endpoints is Section SC-7: Boundary Protection. Within the context of this control, an information system should adhere to the following directives:<br>
<br>
a. Monitors and controls communications at the external boundary of the system and at key internal boundaries within the system; <br>
b. Implements subnetworks for publicly accessible system components that are [Selection: physically; logically] separated from internal organizational networks; and <br>
c. Connects to external networks or information systems only through managed interfaces consisting of boundary protection devices arranged in accordance with an organizational security architecture. <br>
<br>
Specifically, section SC-7(3) states:<br>
<br>
"The organization limits the number of external network connections to the information system. Limiting the number of external network connections facilitates more comprehensive monitoring of inbound and outbound communications traffic."<br>
<br>
I opted to use this section from NIST 800-53 because, given the malicious activity I had encountered, it appeared to be the "lowest hanging fruit" and promised the most significant enhancements to my cloud environment's security posture. Azure's Microsoft Defender for Cloud gave me the following security control recommendations based on NIST 800-53 SC-7(3):


![Screenshot 2023-09-07 190615](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/88345ec8-e132-42e4-a080-a21c8dca21e6)

## Configuring and Hardening Network Security Groups
- From your Azure Account main page go to Network Security Groups.
- For each VM's NSG, add a new rule for inbound connections. Set the priority to the lowest (100) to ensure this rule will be followed first.
- Select the source section, and select IP Addresses, and enter in the addresses from which you will be allowing access to this machine.
- If you are only accessing it from one IP Address, you can select "My IP Address," and Azure will automatically detect and add your current address.<br>

![Screenshot 2023-09-07 200607](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/6fe25e75-d941-45a2-8deb-8582030414d5)


## Enabling Firewall on Azure Key Vault and Blob Storage
- The next step is for us to disable public access to our Azure Key Vault and Blob Storage services.
- Navigate to Key Vault, select the vault connected to your subscription and go to the Networking tab.
- In the Firewalls and virtual network settings, select "disable public access."
- By selecting this option you are configuring the Key Vault to be accessible only from within your virtual network (VNet) or selected virtual networks and not accessible from the public internet. This provides an additional layer of security by restricting access to the Key Vault to resources within your trusted Azure VNets or specific IP ranges you have defined.<br>
<br>

![Screenshot 2023-09-07 192007](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/7c7ec466-ab6a-40f4-97ee-05ae3ebb5b86)<br>

- Now, navigate to Storage Accounts, select the account connected to your subscription and go to the Networking tab again.
- This is similar to the Key Vault, go to the Firewall and network settings and under "public network access" select disabled.<br>

![Screenshot 2023-09-07 192617](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/e6f954d9-0954-4465-9bb9-1f8aca3216e1)<br>

- Next, navigate to the configuration tab and disable "allow blob anonymous access."
- This option prevents public access to containers and blobs within a storage account without proper authentication and authorization.<br>
<br>

  ![Screenshot 2023-09-07 192456](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/641df0f3-9e02-49f1-a9c9-a3d2e08a3b17)

## Configuring Private Endpoint Access to Azure Key Vault and Blob Storage
- Creating a private endpoint for Azure Storage and Azure Key Vault enhances the security and network isolation of these services by allowing you to access them privately within your virtual network (VNet) instead of over the public internet.

**Azure Storage Account**
- Navigate to the Networking tab again, but this time you want to go to private endpoint connections settings.
- Click + to create a new private endpoint and make sure in the basics settings that you select the correct resource group and name it appropriately.<br>

![Screenshot 2023-09-12 220338](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/be48d1c9-a8e4-4f86-bfab-92afb704469a)<br>

- In the resource settings, select "blob" as the target sub-resource group.<br>

![Screenshot 2023-09-07 192732](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/b411ba3e-cb18-46f9-8e86-62eb6e5e6ebe)<br>

- Lastly, go to the virtual network settings and select "dynamically allocate IP Address," and allocate it to the correct VNet associated with your desired resource group. <br>

![Screenshot 2023-09-07 192758](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/0992994b-9445-4476-b083-ea3c663e5e96)<br>
<br>

**Azure Key Vault**
- This same process can be repeated for Azure Key Vault, click + to create a new private endpoint, select the correct resource group and name it appropriately.<br>

![Screenshot 2023-09-12 221130](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/e0d5f1e7-be00-4173-91e0-fc39eba29a53)<br>

- In the resource settings, select "vault" as the target sub-resource group.<br>

![Screenshot 2023-09-12 221221](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/79250461-b422-49a7-8689-8b2186f30f5f)<br>

- Go to the virtual network settings and select "dynamically allocate IP Address," and allocate it to the correct VNet associated with your desired resource group.
- Lastly, go to review and create and click create to finalize your private endpoints. <br>

![Screenshot 2023-09-12 221255](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/acc35748-3fb3-439c-b63f-9aaf115410ce)<br>

## Verifying that you have successfully configured private endpoints
- Once created, we will use `nslookup` to ensure that these private endpoints have been created successfully.
- For the Storage Account, navigate to the Endpoints tab and copy the "Primary Endpoint" address without the http:// prefix and enter the command `nslookup ADDRESS`.
- For the Key Vault, navigate to the Properties tab and copy the "Vault URI" without the http:// prefix and enter the same command with the vault address.
- If your private endpoints have been correctly configured, you will receive back a private IP address such as 10.x.x.x.<br>
![Screenshot 2023-09-07 194757](https://github.com/Lachiecodes/Azure-Cloud-Security-Hardening/assets/138475757/a4aa4563-4ac1-41c8-846c-09929ce15250)
