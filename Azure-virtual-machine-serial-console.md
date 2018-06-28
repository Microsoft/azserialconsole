# Azure's Virtual Machine Serial Console -- Preview 

> [!NOTE] 
> Previews are made available to you on the condition that you agree to the terms of use. For more information, see [Microsoft Azure Supplemental Terms of Use for Microsoft Azure Previews.] (https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/)

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
Windows images on Azure do not have [Special Administrative Console](https://technet.microsoft.com/en-us/library/cc787940(v=ws.10).aspx) (SAC) enabled by default. SAC is supported on server versions of Windows but is not available on client versions (e.g. Windows 10, Windows 8 or Windows 7).

To enable Serial console for Windows Virtual Machines please the following steps: 

1. Connect to your Windows virtual machine via Remote Desktop
2. From an Administrative command prompt run the following commands 
* `bcdedit /ems {current} on`
* `bcdedit /emssettings EMSPORT:1 EMSBAUDRATE:115200`
3. Reboot the system for the SAC console to be enabled

![](https://github.com/Microsoft/azserialconsole/blob/master/images/SerialConsole-PrivatePreviewWindows.gif)

## Windows Commands - CMD 

The table below shows examples of how to accomplish tasks in scenarios where you may need to use SAC to access the VM, such as when you need to troubleshoot RDP connection failures.

SAC has been included in all versions of Windows since Windows Server 2003 but is disabled by default. SAC relies on the `sacdrv.sys` kernel driver, the `Special Administration Console Helper` service (`sacsvr`), and the `sacsess.exe` process. For more information see [Emergency Management Services Tools and Settings](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc787940(v%3dws.10)).

SAC allows you to connect to your running OS via serial port. When you launch CMD from SAC, `sacsess.exe` launches `cmd.exe` within your running OS. You can see that in Task Manager if you RDP to your VM at the same time you are connected to SAC via the serial console feature. The CMD you access via SAC is the same `cmd.exe` you use when connected via RDP. All the same commands and tools are available, including the ability to launch PowerShell from that CMD instance. That is a major difference between SAC and the Windows Recovery Environment (WinRE) in that SAC is letting you manage your running OS, where WinRE boots into a different, minimal OS. While Azure VMs do not support the ability to access WinRE, with the serial console feature, Azure VMs can be managed via SAC.

Because SAC is limited to an 80x24 screen buffer with no scrollback, add `| more` to commands to display the output one page at a time. Use `<spacebar>` to see the next page, or `<enter>` to see the next line.  

`SHIFT+INSERT` is the paste shortcut for the serial console window.

Because of SAC's limited screen buffer, longer commands may be easier to type out in a local text editor and then pasted into SAC.

Task | Command | Notes 
-----|---------|----------
Verify RDP is enabled | `reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections`<br><br>`reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections` | The second key (under \Policies) will only exist if the relevant group policy setting is configured.
Enable RDP | `reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0`<br><br>`reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fDenyTSConnections /t REG_DWORD /d 0` | The second key (under \Policies) would only be needed if the relevant group policy setting had been configured. Value will be rewritten at next group policy refresh if it is configured in group policy.
View service state | `sc query termservice` | 
View service logon account | `sc qc termservice` |
Set service logon account | `sc config termservice obj= "NT Authority\NetworkService"` | A space is required after the equals sign.
Set service start type | `sc config termservice start= demand`| A space is required after the equals sign. Possible start values include `boot`, `system`, `auto`, `demand`, `disabled`, `delayed-auto`.
Set service dependencies | `sc config termservice depend= RPCSS` | A space is required after the equals sign.
Start service | `net start termservice`<br><br>or<br><br>`sc start termservice` | 
Stop service | `net stop termservice`<br><br>or<br><br>`sc stop termservice` |  
Show NIC properties | `netsh interface show interface` |
Show IP properties | `netsh interface ip show config` | 
Enable NIC | `netsh interface set interface name="<interface name>" admin=enabled`
Set NIC to use DHCP | `netsh interface ip set address name="<interface name>" source=dhcp` | Azure VMs should always be configured in the guest OS to use DHCP to obtain an IP address. The Azure static IP setting still uses DHCP to give the static IP to the VM.
Ping | `ping 8.8.8.8` |
Port ping | Install the telnet client<br><br>`dism /online /Enable-Feature /FeatureName:TelnetClient`<br><br>Test connectivity<br><br>`telnet bing.com 80`<br><br>To remove the telnet client<br><br>`dism /online /Disable-Feature /FeatureName:TelnetClient` | When limited to methods available in Windows by default, PowerShell can be a better approach for testing port connectivity. See the PowerShell section below for examples.
Test DNS name resolution | `nslookup bing.com` | 
Show Windows firewall rule | `netsh advfirewall firewall show rule name="Remote Desktop - User Mode (TCP-In)"`
Disable Windows firewall | `netsh advfirewall set allprofiles state off` | 
Create local user account | `net user /add <username> <password>` |
Add local user to local group | `net localgroup Administrators <username> /add` | 
Verify user account is enabled | `net user <username> | find /i "active"` | Note that Azure VMs created from generalized image will have the local administrator account renamed to the name specified during VM provisioning. So it will usually not be `Administrator`.
Enable user account | `net user <username> /active:yes` | 
View user account properties | `net user <username>` | Example lines of interest from a local admin account<br>`Account active Yes`<br>`Account expires Never`<br>`Password expires Never`<br>`Workstations allowed All`<br>`Logon hours allowed All`<br>`Local Group Memberships *Administrators`
View local groups | `net localgroup` |
Query event log errors | `wevtutil qe system /c:10 /f:text /q:"Event[System[Level=2]]" | more` | Change `/c:10` to the desired number of events to return, or move it to return all events matching the filter.
Query event log by Event ID | `wevtutil qe system /c:1 /f:text /q:"Event[System[EventID=11]]" | more` |
Query event log by Event ID and Provider | `wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11]]" | more` |
Query event log by Event ID and Provider for the last 24 hours | `wevtutil qe system /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11 and TimeCreated[timediff(@SystemTime) <= 86400000]]]"` | Use `604800000` to look back 7 days instead of 24 hours.
Query event log by Event ID, Provider, and EventData in the last 7 days | `wevtutil qe security /c:1 /f:text /q:"Event[System[Provider[@Name='Microsoft-Windows-Security-Auditing'] and EventID=4624 and TimeCreated[timediff(@SystemTime) <= 604800000]] and EventData[Data[@Name='TargetUserName']='<username>']]" | more` |
List installed software | `wmic product get Name,InstallDate | sort /r | more` | The `sort /r` sorts descending by install date to make it easy to see what was recently installed. Use `<spacebar>` to advance to the next page of output, or `<enter>` to advance one line.
Uninstall software | `wmic path win32_product where name="<name>" call uninstall` | Replace `<name>` with the name returned in the above command for the application you want to remove.
Get file version | `wmic datafile where "drive='C:' and path='\\windows\\system32\\drivers\\' and filename like 'netvsc%'" get version /format:list` | This example returns the file version of the virtual NIC driver, which is netvsc.sys, netvsc63.sys, or netvsc60.sys depending on the Windows version.
Scan for system file corruption | `sfc /scannow` | See also [Repair a Windows Image](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/repair-a-windows-image).
Scan for system file corruption | `dism /online /cleanup-image /scanhealth` | See also [Repair a Windows Image](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/repair-a-windows-image).
Remove non-present PNP devices | `%windir%\System32\RUNDLL32.exe %windir%\System32\pnpclean.dll,RunDLL_PnpClean /Devices /Maxclean` |
Export file permissions to text file | `icacls %programdata%\Microsoft\Crypto\RSA\MachineKeys /t /c > %temp%\MachineKeys_permissions_before.txt`
Save file permissions to ACL file | `icacls %programdata%\Microsoft\Crypto\RSA\MachineKeys /save %temp%\MachineKeys_permissions_before.aclfile /t` | 
Restore file permissions from ACL file | `icacls %programdata%\Microsoft\Crypto\RSA /save %temp%\MachineKeys_permissions_before.aclfile /t` | Note that the path when using `/restore` needs to be the parent folder of the folder you specified when using `/save`. In this example, `\RSA` is the parent of the `\MachineKeys` folder specified in the `/save` example above.
Take NTFS ownership of a folder | `takeown /f %programdata%\Microsoft\Crypto\RSA\MachineKeys /a /r` | 
Grant NTFS permissions to a folder recursively | `icacls C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys /t /c /grant "BUILTIN\Administrators:(F)"` | 
Show IPSec configuration | `netsh nap client show configuration` | 
Force group policy update | `gpupdate /force /wait:-1` | 
Show OS version | `ver`<br><br>`wmic os get caption,version,buildnumber /format:list`<br><br>`systeminfo | find /i "os name"`<br>`systeminfo | findstr /i /r "os.*version.*build"`<br><br>
Restart Windows | `shutdown /r /t 0` | Adding `/f` will force running applications to close without warning users.
Detect Safe Mode boot | `bcdedit /enum | find /i "safeboot"` |
View OS install date | `systeminfo | find /i "original"`<br><br>or<br><br>`wmic os get installdate` |
View last boot time | `systeminfo | find /i "system boot time"` |
View time zone | `systeminfo | find /i "time zone"`<br><br>or<br><br>`wmic timezone get caption,standardname /format:list` | 

## Windows Commands - PowerShell

WARNING: There is a known issue with pasting text into PowerShell in SAC where an extra character may be introducted into the text being pasted. Typically this occurs when pasting consecutive characters that are the same, e.g. "boot" is pasted as "booot" and "install" is pasted as "installl". The issue does not exist when pasting text in CMD in SAC, only when using PowerShell in SAC, and only on Windows Server 2016 and later.

Please carefully review text pasted in PowerShell in SAC for extra characters, or use CMD instead where this issue does not occur.

To run PowerShell in SAC, after you reach a CMD prompt, type:

`powershell <enter>`

Task | Command | Notes 
-----|---------|----------
Verify RDP is enabled | `get-itemproperty -path 'hklm:\system\curRentcontrolset\control\terminal server' -name 'fdenytsconNections'`<br><br>`get-itemproperty -path 'hklm:\software\policies\microsoft\windows nt\terminal services' -name 'fdenytsconNections'` | The second key (under \Policies) will only exist if the relevant group policy setting is configured.
Enable RDP | `set-itemproperty -path 'hklm:\system\curRentcontrolset\control\terminal server' -name 'fdenytsconNections' 0 -type dword`<br><br>`set-itemproperty -path 'hklm:\software\policies\microsoft\windows nt\terminal services' -name 'fdenytsconNections' 0 -type dword` | The second key (under \Policies) would only be needed if the relevant group policy setting had been configured. Value will be rewritten at next group policy refresh if it is configured in group policy.
View service details | `get-wmiobject win32_service -filter "name='termservice'" | fl Name,DisplayName,State,StartMode,StartName,PathName,ServiceType,Status,ExitCode,ServiceSpecificExitCode,ProcessId` | `Get-Service` can be used but doesn't include the service logon account. `Get-WmiObject win32-service` does.
Set service logon account | `(get-wmiobject win32_service -filter "name='termservice'").Change($null,$null,$null,$null,$null,$false,'NT Authority\NetworkService')` | When specifying a service account other than `NT AUTHORITY\LocalService`, `NT AUTHORITY\NetworkService`, or `LocalSystem` you need to specify the account's password as the last (8th) argument after the account name.
Set service startup type | `set-service termservice -startuptype Manual`| `Set-service` accepts `Automatic`, `Manual`, or `Disabled` for startup type.
Set service dependencies | `Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\TermService' -Name DependOnService -Value @('RPCSS','TermDD')` | 
Start service | `start-service termservice` | 
Stop service | `stop-service termservice` |  
Show NIC properties | `get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} | fl status,name,ifdesc,macadDresS,driverversion,MediaConNectState,MediaDuplexState`<br><br>`get-wmiobject win32_networkadapter -filter "servicename='netvsc'" | fl netenabled,name,macaddress` | `Get-NetAdapter` is available in 2012+, for 2008R2 use `Get-WmiObject`.
Show IP properties | `get-wmiobject Win32_NetworkAdapterConfiguration -filter "ServiceName='netvsc'" | fl DNSHostName,IPAddress,DHCPEnabled,IPSubnet,DefaultIPGateway,MACAddress,DHCPServer,DNSServerSearchOrder` | 
Enable NIC | `get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} | enable-netadapter`<br><br>`(get-wmiobject win32_networkadapter -filter "servicename='netvsc'").enable()` | `Get-NetAdapter` is available in 2012+, for 2008R2 use `Get-WmiObject`.
Set NIC to use DHCP | `get-netadapter | where {$_.ifdesc.startswith('Microsoft Hyper-V Network Adapter')} | Set-NetIPInterface -DHCP Enabled`<br><br>`(get-wmiobject Win32_NetworkAdapterConfiguration -filter "ServiceName='netvsc'").EnableDHCP()` | `Get-NetAdapter` is available on 2012+. For 2008R2 use `Get-WmiObject`. Azure VMs should always be configured in the guest OS to use DHCP to obtain an IP address. The Azure static IP setting still uses DHCP to give the IP to the VM.
Ping | `test-netconnection`<br><br>`get-wmiobject Win32_PingStatus -Filter 'Address="8.8.8.8"' | ft -a IPV4Address,ReplySize,ResponseTime` | `Test-Netconnection` without any parameters will try to ping `internetbeacon.msedge.net`. It is available on 2012+. For 2008R2 use `Get-WmiObject` as in the second example.
Port Ping | `test-netconnection -ComputerName bing.com -Port 80`<br><br>`(new-object Net.Sockets.TcpClient).BeginConnect('bing.com','80',$null,$null).AsyncWaitHandle.WaitOne(300)` | `Test-NetConnection` is available on 2012+. For 2008R2 use `Net.Sockets.TcpClient`
Test DNS name resolution | `resolve-dnsname bing.com`<br><br>`[System.Net.Dns]::GetHostAddresses('bing.com')` | `Resolve-DnsName` is available on 2012+. For 2008R2 use `System.Net.DNS`.
Show Windows firewall rule by name | `get-netfirewallrule -name RemoteDesktop-UserMode-In-TCP` | 
Show Windows firewall rule by port | `get-netfirewallportfilter | where {$_.localport -eq 3389} | foreach {Get-NetFirewallRule -Name $_.InstanceId} | format-list Name,Enabled,Profile,Direction,Action`<br><br>`(new-object -ComObject hnetcfg.fwpolicy2).rules | where {$_.localports -eq 3389 -and $_.direction -eq 1} | format-table Name,Enabled` | `Get-NetFirewallPortFilter` is available on 2012+. For 2008R2 use the `hnetcfg.fwpolicy2` COM object. 
Disable Windows firewall | `Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False` | `Set-NetFirewallProfile` is available on 2012+. For 2008R2 use `netsh advfirewall` as referenced in the CMD section above.
Create local user account | `new-localuser <name>`|
Verify user account is enabled | `(get-localuser | where {$_.SID -like "S-1-5-21-*-500"}).Enabled`<br><br>`(get-wmiobject Win32_UserAccount -Namespace "root\cimv2" -Filter "SID like 'S-1-5-%-500'").Disabled` | `Get-LocalUser` is available on 2012+. For 2008R2 use `Get-WmiObject`. This example shows the builtin local administrator account, which always has SID `S-1-5-21-*-500`. Note that Azure VMs created from generalized image will have the local administrator account renamed to the name specified during VM provisioning. So it will usually not be `Administrator`.
Add local user to local group | `add-localgroupmember -group Administrators -member <username>` |
Enable local user account | `get-localuser | where {$_.SID -like "S-1-5-21-*-500"} | enable-localuser` | This example enables the builtin local administrator account, which always has SID `S-1-5-21-*-500`. Note that Azure VMs created from generalized image will have the local administrator account renamed to the name specified during VM provisioning. So it will usually not be `Administrator`.
View user account properties | `get-localuser | where {$_.SID -like "S-1-5-21-*-500"} | format-list *`<br><br>`get-wmiobject Win32_UserAccount -Namespace "root\cimv2" -Filter "SID like 'S-1-5-%-500'" | fl Name,Disabled,Status,Lockout,Description,SID` | `Get-LocalUser` is available on 2012+. For 2008R2 use `Get-WmiObject`. This example shows the builtin local administrator account, which always has SID `S-1-5-21-*-500`.
View local groups | `(get-localgroup).name | sort`<br><br>`(get-wmiobject win32_group).Name | sort` | `Get-LocalUser` is available on 2012+. For 2008R2 use `Get-WmiObject`.
Query event log errors | `get-winevent -logname system -maxevents 1 -filterxpath "*[System[Level=2]]" | more` | Change `/c:10` to the desired number of events to return, or move it to return all events matching the filter.
Query event log by Event ID | `get-winevent -logname system -maxevents 1 -filterxpath "*[System[EventID=11]]" | more` |
Query event log by Event ID and Provider | `get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11]]" | more` |
Query event log by Event ID and Provider for the last 24 hours | `get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Hyper-V-Netvsc'] and EventID=11 and TimeCreated[timediff(@SystemTime) <= 86400000]]]"` | Use `604800000` to look back 7 days instead of 24 hours. |
Query event log by Event ID, Provider, and EventData in the last 7 days | `get-winevent -logname system -maxevents 1 -filterxpath "*[System[Provider[@Name='Microsoft-Windows-Security-Auditing'] and EventID=4624 and TimeCreated[timediff(@SystemTime) <= 604800000]] and EventData[Data[@Name='TargetUserName']='<username>']]" | more` |
List installed software | `get-wmiobject win32_product | select installdate,name | sort installdate -descending | more` |
Uninstall software | `(get-wmiobject win32_product -filter "Name='<name>'").Uninstall()` |
Show OS version | `get-wmiobject win32_operatingsystem | format-list caption,version,buildnumber`<br><br>
View OS install date | `(get-wmiobject win32_operatingsystem).converttodatetime((get-wmiobject win32_operatingsystem).installdate)` |
View last boot time | `(get-wmiobject win32_operatingsystem).lastbootuptime` |
View Windows uptime | `"{0:dd}:{0:hh}:{0:mm}:{0:ss}.{0:ff}" -f ((get-date)-(get-wmiobject win32_operatingsystem).converttodatetime((get-wmiobject win32_operatingsystem).lastbootuptime))` | Returns uptime as `<days>:<hours>:<minutes>:<seconds>:<milliseconds>`, for example `49:16:48:00.00`. 
Restart Windows | `restart-computer` | Adding `-force` will force running applications to close without warning users.
Get file version | `(get-childitem $env:windir\system32\drivers\netvsc*.sys).VersionInfo.FileVersion` | This example returns the file version of the virtual NIC driver, which is named netvsc.sys, netvsc63.sys, or netvsc60.sys depending on the Windows version.
Download and extract file | `$path='c:\bin';md $path;cd $path;(new-object net.webclient).downloadfile( ('htTp:/'+'/download.sysinternals.com/files/SysinternalsSuite.zip'),"$path\SysinternalsSuite.zip");(new-object -com shelL.apPlication).namespace($path).CopyHere( (new-object -com shelL.apPlication).namespace("$path\SysinternalsSuite.zip").Items(),16)` | 
Instance metadata | `$im = invoke-restmethod -headers @{"metadata"="true"} -uri http://169.254.169.254/metadata/instance?api-version=2017-08-01 -method get`<br>`$im | convertto-json` |
OS Type (Instance Metadata) | `$im.Compute.osType` |
Location (Instance Metadata) | `$im.Compute.Location` |
Size (Instance Metadata) | `$im.Compute.vmSize` |
VM ID (Instance Metadata) | `$im.Compute.vmId` |
VM Name (Instance Metadata) | `$im.Compute.name` |
Resource Group Name (Instance Metadata) | `$im.Compute.resourceGroupName` |
Subscription ID (Instance Metadata) | `$im.Compute.subscriptionId` |
Tags (Instance Metadata) | `$im.Compute.tags` |
Placement Group ID (Instance Metadata) | `$im.Compute.placementGroupId` |
Platform Fault Domain (Instance Metadata) | `$im.Compute.platformFaultDomain` |
Platform Update Domain (Instance Metadata) | `$im.Compute.platformUpdateDomain` |
IPv4 Private IP Address (Instance Metadata) | `$im.network.interface.ipv4.ipAddress.privateIpAddress` |
IPv4 Public IP Address (Instance Metadata) | `$im.network.interface.ipv4.ipAddress.publicIpAddress` |
IPv4 Subnet Address / Prefix (Instance Metadata) | `$im.network.interface.ipv4.subnet.address`<br><br>`$im.network.interface.ipv4.subnet.prefix` |
IPv6 IP Address (Instance Metadata) | `$im.network.interface.ipv6.ipAddress` |
MAC Address (Instance Metadata) | `$im.network.interface.macAddress` |

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
Hitting enter after the connection banner does not show a login prompt | [Hitting enter does nothing](./Known_issues/Hitting_enter_does_nothing.md)
Only health information is shown when connecting to a Windows VM | [Only Windows health info presented](./Known_issues/Windows_Health_Info.md)

## Availability 
The current preview is available in **West US Central** and **West Europe** regions of Azure.

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
