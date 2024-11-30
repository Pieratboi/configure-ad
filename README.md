<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

# Active Directory Lab - Comprehensive Setup, Deployment, and Group Policy Exploration
This documentation covers the comprehensive setup and management of an Active Directory environment using Azure Virtual Machines (VMs). The lab includes preparing Azure VMs, deploying and configuring Active Directory (AD), and exploring advanced Group Policy configurations for real-world applications.

Active Directory is a cornerstone of enterprise IT infrastructure, offering centralized domain management, user authentication, and policy enforcement. This lab demonstrates foundational and advanced skills in managing AD, including creating users, configuring permissions, applying Group Policies, and auditing security logs.

---

## Table of Contents
1. [Preparing Azure VMs for Active Directory](#preparing-azure-vms-for-active-directory)
2. [Deploying and Configuring Active Directory](#deploying-and-configuring-active-directory)
3. [Exploring Group Policy Applications](#exploring-group-policy-applications)
    - [Remote Desktop Access](#remote-desktop-access)
    - [Account Management Policies](#account-management-policies)
    - [Advanced Group Policies](#advanced-group-policies)
4. [Best Practices in AD Management](#best-practices-in-ad-management)
5. [Troubleshooting](#troubleshooting)
6. [Licensing Information](#licensing-information)
7. [Contact Information](#contact-information)

---

## Preparing Azure VMs for Active Directory

### Domain Controller Setup
1. **Create a Resource Group**:
   - Navigate to the Azure Portal and create a new Resource Group to organize your resources.

2. **Set Up a Virtual Network and Subnet**:
   - Define a Virtual Network and a Subnet to ensure all VMs can communicate securely.

3. **Deploy the Domain Controller (DC-1)**:
   - Create a VM running **Windows Server 2022** named **DC-1**.
     - Username: `labuser`
     - Password: `Cyberlab123!`
   - Assign a static private IP address to DC-1's NIC. (DC-1 -> Network Settings -> Virtual NiC -> defaultipconfig -> Private IP static)

4. **Prepare DC-1**:
   - Log into the VM and disable the Windows Firewall temporarily (testing purposes only).

### Client VM Setup
1. **Deploy Client-1**:
   - Create a VM running **Windows 10** named **Client-1**.
     - Username: `labuser`
     - Password: `Cyberlab123!`
   - Attach it to the same Virtual Network and Subnet as **DC-1**.

2. **Configure Networking for Client-1**:
   - Set **Client-1's DNS settings** to DC-1's private IP address.
   - Restart **Client-1** from the Azure Portal.

3. **Verify Connectivity**:
   - Log into **Client-1**, open PowerShell, and verify connectivity to DC-1 using `ping <DC-1 private IP>`.
   - Run `ipconfig /all` to confirm the DNS settings reflect DC-1's private IP address.

![image](https://github.com/user-attachments/assets/6ffe26fc-6e4c-4525-abfd-97f93ecd670a)
(Resource Group housing our Virtual Network, Virtual Machines [Domain Controller & Client])

![image](https://github.com/user-attachments/assets/c40604e8-7c3e-4eaa-ba13-22c402ec8c9a)

(client1 DNS set to DC-1's private IP)

---

## Deploying and Configuring Active Directory

![image](https://github.com/user-attachments/assets/a52b9503-0019-47e3-bb04-0513ee05a9b4)

### Installing Active Directory and Setting Up the Domain
1. **Install Active Directory Domain Services**:
   - Log into **DC-1**, open **Server Manager**, and install the **Active Directory Domain Services** role.

2. **Promote DC-1 as a Domain Controller**:
   - Set up a new forest (e.g., `mydomain.com`).
   - Restart and log back into **DC-1** as `mydomain.com\labuser`.

### Creating Administrative Accounts
1. **Organize Users**:
   - Open **Active Directory Users and Computers (ADUC)**.
   - Create Organizational Units (OUs):
     - `_EMPLOYEES`
     - `_ADMINS`

2. **Create a Domain Admin**:
   - Add a user named `jane_admin` (Password: `Cyberlab123!`) to the **Domain Admins** security group.

3. **Use the Domain Admin Account**:
   - Log out and log back in as `mydomain.com\jane_admin` for administrative tasks.

### Joining Client-1 to the Domain
1. **Update DNS Settings**:
   - Ensure **Client-1's DNS settings** point to DC-1.
   - Restart **Client-1**.

2. **Join the Domain**:
   - Log into **Client-1** as the local admin (`labuser`) and join it to `mydomain.com`.
   - Restart **Client-1**.

3. **Verify in ADUC**:
   - Log into **DC-1** and confirm **Client-1** appears in **ADUC**.
   - Move **Client-1** to a new OU named `_CLIENTS`.

![image](https://github.com/user-attachments/assets/40af679d-fb4b-4dc4-9369-2980c33ab7a6)

(Client-1 PC recognized in mydomain.com, Jane Doe member of Domain Admins SecGrp)

![image](https://github.com/user-attachments/assets/492c97db-da3a-4af1-8acd-74a8300ee861)

(Client-1 member of mydomain.com)

---

## Exploring Group Policy Applications

### Remote Desktop Access
1. **Enable Remote Desktop for Domain Users**:
   - Log into **Client-1** as `mydomain.com\jane_admin`.
   - Open **System Properties** and configure Remote Desktop to allow access for **Domain Users**.

2. **Test User Access**:
   - Log out and attempt to log into **Client-1** using a non-administrative user account.

### Account Management Policies
1. **Configure Account Lockout Policies**:
   - Log into **DC-1**, open **Group Policy Management**.
   - Modify the **Default Domain Policy** to enforce account lockout after 5 failed attempts.
   - Test by attempting 6 incorrect logins with a test account.

2. **Unlock and Reset Accounts**:
   - Observe locked accounts in **ADUC**, unlock them, reset their passwords, and verify login success.

### Advanced Group Policies
1. **Mapped Drives for Employees**:
   - Use Group Policy Preferences to map a shared drive for all users in `_EMPLOYEES`.

2. **Default Browser Configuration**:
   - Create a policy to enforce a default browser for all domain-joined devices.

3. **Password Policies**:
   - Set password complexity and expiration policies in the **Default Domain Policy**.

4. **Software Deployment**:
   - Deploy essential software (e.g., antivirus) to domain-joined machines using Group Policy.

5. **Audit Logon Events**:
   - Enable auditing of logon events to monitor user activity for security purposes.

---

## Best Practices in AD Management

- Always maintain a structured OU hierarchy to simplify Group Policy management.
- Regularly monitor security logs to identify unauthorized access attempts.
- Back up Active Directory data to prevent data loss during critical failures.
- Automate repetitive administrative tasks using PowerShell scripts.

---

## Troubleshooting

1. **Issue**: Unable to ping **DC-1** from **Client-1**.  
   - **Cause**: Incorrect DNS settings or firewall is enabled on DC-1.  
   - **Solution**: Verify DNS settings on Client-1 and disable the firewall on DC-1.

2. **Issue**: Group Policies are not applied on Client-1.  
   - **Cause**: Group Policy updates are delayed.  
   - **Solution**: Use `gpupdate /force` on Client-1 to apply changes immediately.

3. **Issue**: Accounts do not lock after failed login attempts.  
   - **Cause**: Account Lockout Policy is not configured correctly.  
   - **Solution**: Verify the policy settings in **Group Policy Management** and test again.

---

## Licensing Information
Active Directory and Windows Server are licensed under Microsoft’s licensing agreements. For more details, visit [Microsoft Licensing](https://www.microsoft.com/licensing).

---

## Contact Information
For questions or feedback, feel free to reach out:  
- **GitHub**: https://github.com/Pieratboi  
- **LinkedIn**: <a href="https://www.linkedin.com/in/rafael-razapov-60391a2b8/?trk=opento_sprofile_topcard"> Rafael Razapov </a> 
