# Azure's Virtual Machine Serial Console -- Early Access Preview (ONLY AVAILABLE IN WEST CENTRAL US REGION)

> [!NOTE] 
> Previews are made available to you on the condition that you agree to the terms of use. For more information, see [Microsoft Azure Supplemental Terms of Use for Microsoft Azure Previews.] (https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/)
> ACCESS VIA SERIAL CONSOLE SHOULD BE LIMITED TO DEV-TEST VIRTUAL MACHINES. WE STRONGLY RECOMMEND NOT TO USE THIS FOR ANY PRODUCTION VIRTUAL MACHINE.
>

Azure's Virtual Machine Serial Console provides access to a text-based console for Linux and Windows Virtual Machines on Azure. This serial connection is to COM1 serial port of the virtual machine and provides access to the virtual machine regardless of that virtual machine's network / operating system state. Access to Serial Console for a virtual machine can be done only via Azure Portal and for those who have VM Contributor or above access. 

### Important information
Currently this service is in **preview** and access to Serial Console for Virtual Machines is limited to white-listed subscriptions who are participating in early access preview and only for virtual machines available in West Central US region.

## Accessing Serial Console
Serial Console for Virtual Machines is only accessible via [Azure portal](https://portal.azure.com). Below are the steps to access Serial Console for Virtual Machines via portal 

1. Open the Virtual Machine that you want to access in Azure Portal 
2. Scroll down onto the Support + Troubleshooting section 
3. Click on Serial Console (Preview) option to access serial console 

![](https://github.com/Microsoft/azserialconsole/blob/master/images/SerialConsole-LinuxAccess.gif)

> [!NOTE] 
> Only subscriptions that are white-listed as a part of early access preview will be able to see Serial Console section in the portal. Please also note that serial console requires a local user with a password configured. At this time, VMs only configured with SSH public key will not have access to serial console. To create a local user with password follow [VM Access Extension](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-vmaccess-extension) and create local user with password.

## Serial Console  Security 

### Access Security 
Access to Serial console is limited to users who have [VM Contributors](https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-built-in-roles#virtual-machine-contributor) or above access to the Virtual Machine. If your AAD tenant requires Multi-Factor Authentication then access to serial console will also need MFA as its access is via [Azure Portal](https://portal.azure.com).

### Channel Security
All data is sent back and forth is encrypted on the wire.

### Audit logs
All access to Serial Console are currently logged in the [boot diagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/boot-diagnostics) logs of the virtual machine. Access to these logs are owned and controlled by the Azure Virtual Machine administrator.  

>[!CAUTION] While no access passwords for the console are logged, if commands run within the console contain or output passwords, secrets, user names or any other form of Personally Identifiable Information (PII), those will be written to the Virtual Machine boot diagnostics logs, along with all other visible text, as part of the implementation of the serial console's scrollback functionality. These logs are circular and only individuals with read permissions to the diagnostics storage account have access to them, however we recommend following the best practice of using the SSH console for anything that may involve secrets and/or PII. 

### Concurrent usage
If a user is connected to Serial Console and another user successfully requests access to that same virtual machine, the first user will be disconnected and the second user connected in a manner akin to the first user standing up and leaving the physical console and a new user sitting down.

>[!CAUTION] This means that the user who gets disconnected will not be logged out! The ability to enforce a logout upon disconnect (via SIGHUP or similar mechanism) is still in development.

### Disable feature
The serial console functionality can be deactivated for specific VMs by disabling that VM's boot diagnostics setting.

## Common Scenarios for accessing Serial Console 
Scenario          | Actions in Serial Console                |  OS Applicability 
------------------|:----------------------------------------:|------------------:
Broken FSTAB file | Enter key to continue and fix fstab file using a text editor | Linux 
Incorrect firewall rules | Access Serial Console and fix iptables or Windows firewall rules | Linux/Windows 
Filesystem corruption/check | Access Serial Console and recover filesystem | Linux/Windows 
SSH/RDP configuration issues | Access Serial Console and change settings | Linux/Windows 
Network lock down system| Access Serial Console via portal to manage system | Linux/Windows 
Interacting with bootloader | Access GRUB/BCD via serial console | Linux/Windows 

## Accessing Serial Console for Linux ( Distro Specific Scenarios ) 
In order for Serial Console to function properly, the guest operating system must be configured to read and write console messages to the serial port. Most [Endorsed Azure Linux Distributions](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/endorsed-distros) have serial console configured by default. Just by clicking in the portal on the Serial console section will provide access to the console. 

### Access for RedHat 
RedHat Images available on Azure have serial console enabled by default. Single User Mode in Red Hat requires root user to be enabled, which is disabled by default. If you have a need to enable Single user mode, use the following instructions 

1. Login to the Red Hat system via SSH
2. Enable password for root user 
 * `passwd root` ( set a strong root password )
3. Ensure root user can only login via ttyS0
 * `edit /etc/ssh/sshd_config` and ensure PermitRootLogin is set to no
 * `edit /etc/securetty file` to only allow logins via ttyS0 

Now if the system boots into single user mode you can login via root password.

### Access for Ubuntu 
Ubuntu images available on Azure have serial console enabled by default. If the system boots into Single User Mode you can access without additional credentials. 

### Access for CoreOS
CoreOS images available on Azure have serial console enabled by default. If required system can be booted into Single User Mode via changing GRUB parameters and adding coreos.autologin=ttyS0 would enable core user to login and available in Serial Console. 

### Access for SUSE
SLES images available on Azure currently do not have serial console enabled by default. To enable serial console on SLES please follow the [KB article](https://www.novell.com/support/kb/doc.php?id=3456486). 

### Access for CentOS
CentOS images available on Azure have serial console enabled by default. For Single User Mode please follow instructions similar to Red Hat Images above. 

### Access for Oracle Linux
Oracle Linux images available on Azure have serial console enabled by default. For Single User Mode please follow instructions similar to Red Hat Images above.

### Access for Custom Linux Image
To enable Serial Console for your custom Linux VM image, enable console in /etc/inittab to run a terminal on ttyS0. Below is an example to add this in the inittab file 

`S0:12345:respawn:/sbin/agetty -L 115200 console vt102` 

## Accessing Serial Console for Windows 
Windows images on Azure do not have [Special Administrative Console](https://technet.microsoft.com/en-us/library/cc787940(v=ws.10).aspx) (SAC) enabled by default. To enable Serial console for Windows Virtual Machines please the following steps: 

1. Connect to your Windows virtual machine via Remote Desktop
2. From an Administrative command prompt run the following commands 
* `bcdedit /ems {current} on`
* `bcdedit /emssettings EMSPORT:1 EMSBAUDRATE:115200`
3. Reboot the system for the SAC console to be enabled

![](https://github.com/Microsoft/azserialconsole/blob/master/images/SerialConsole-PrivatePreviewWindows.gif)

## Errors
Most errors are transient in nature and retrying connection address these. Below table shows a list of errors and mitigation 

Error                            |   Mitigation 
---------------------------------|:--------------------------------------------:|
Error 500 Message: Unable to determine the resource group for the boot diagnostics storage account | Ensure that that user has access to boot diagnostics storage account for the said VM 
Error 500 Message: Server encountered an internal error | Please retry, if the issue persists please contact azserialhelp@microsoft.com

## Known Issues 
As we are still in the early stages for Serial Console access, we are working through some known issues, below is the list of these with possible workarounds 

Issue                           |   Mitigation 
---------------------------------|--------------------------------------------|
There is no option with VMSS instance Serial Console | At the time of preview we do not support access to serial console for VMSS instances 
There is no option to enable/disable Serial Console | We are working on this functionality and will come in our upcoming releases
You receive the following error even though you already have at least VM Contributor role permission to the VM: "The serial console connection to the VM encountered an error: 'Forbidden' (403) - You do not have the required permissions to use this VM serial console. Please make sure you have at least VM Contributor role permissions." | Currently there is a known issue where, even if you are an administrator for the subscription and show up as having the Owner role, you need to explicitly give your account the Owner role to avoid the 'Forbidden' error. In the Azure portal search field at the top right, search for **Subscriptions** and then select **Subscriptions** in the results to open it. Select the relevant subscription, then select **Access control (IAM)**. Click **Add**. For **Role** select **Owner**, and in the **Select** field search for your user account, select it, then click **Save** at the bottom.
Error 400-Bad Request when targeting VMs with name containing multiple hyphens | Target a virtual machine with zero or one hyphens
Opening the serial console causes the portal to initially scroll to the left | Scroll portal back over to the right
Error _Unable to determine the resource group for the boot diagnostics storage account '<account>'. Verify that boot diagnostics is enabled for this VM and you have access to this storage account._ For VM in subscriptions with +1000 objects | Target a virtual machine in a subscription with less objects.

## Availability 
The current Preview is available in **West US Central** Region of Azure. 

## FAQ
1. When will Serial Console be made available in other regions?
* We are working on scaling and stablizing the service. In addition we also need to add additional functionality for this service. We will keep the community posted on updates on when we can expand to other Azure regions 
2. How can I send feedback?
* Please provide feedback as an issue by going to aka.ms/serialconsolefeedback . Alternatively(less preferred) Send feedback via azserialhelp@microsoft.com or in the virtual machine category of http://feedback.azure.com 
3. I am not a part of early access preview but I want to participate, how can I?
* We will be opening access to more subscriptions shortly.
4. I get an Error "Existing console has conflicting OS type "Windows" with the requested OS type of Linux
* This is a know issue to fix this, simply open [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) in bash mode and retry.

## Next Steps
Learn more about [bootdiagnostics](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/boot-diagnostics)
