---
title: Windows Forensics 1/2
markmap:
    colorFreezeLevel: 2
    initialExpandLevel: 2
---
## Malware Persistence
### AutoStart locations
- NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run
- NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\RunOnce
- SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer\Run
- SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
- SOFTWARE\Microsoft\Windows\CurrentVersion\Run
- SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit
- %AppData%\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
- SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run
- SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunOnce
- SYSTEM\\<CurrentControlSet\>\Services - (if Start == 0x02 -> start at boot)
### Service Creation/Replacement
#### HKML\SYSTEM\CurrentControlSet\Services
- Start with 0x02 : Automatic
- Start with 0x00 : Boot start of a device driver
#### Links
- [Mandiant APT1](https://contagiodump.blogspot.com/2013/03/mandiant-apt1-samples-categorized-by.html)
- [Peering into GlassRAT](https://paper.seebug.org/papers/APT/APT_CyberCriminal_Campagin/2015/2015.11.23.PEERING_INTO_GLASSRAT/GlassRAT-final.pdf)
- [Stealthy Malware Persistence](https://isc.sans.edu/diary/Wipe+the+drive+Stealthy+Malware+Persistence+-+Part+2/15406)
### Scheduled Tasks
- C:\Windows\Tasks
- C:\Windows\System32\Tasks
- C:\Windows\Schedlgu.txt - XP only
- If TaskScheduler & a process have the same PPID => process has been launched by the taskscheduler
### DLL Hijacking
#### Links
- [Malware Persistence without the Windows Registry](https://cloud.google.com/blog/topics/threat-intelligence/malware-persistence-windows-registry/)
- [DLL Search order](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order)
- [Phantom DLL](https://www.hexacorn.com/blog/2013/12/08/beyond-good-ol-run-key-part-5/)
- [What The Fxsst](https://web.archive.org/web/20211008221934/https://www.fireeye.com/blog/threat-research/2011/06/fxsst.html)
- [NetTraveler](https://www.proofpoint.com/us/threat-insight/post/nettraveler-apt-targets-russian-european-interests)
- [DLL rules in AppLocker](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/dll-rules-in-applocker)
#### Hunting
- File System analyse: search .dll/.exe in unusual path
- Memory analyse: search DLL loaded from wrong location
- [Hunting with Velociraptor](https://docs.velociraptor.app/blog/2021/2021-02-02-detecting-dll-hijacking-with-vql-e9a735354257/)
### WMI Event Consumers
#### Info
- Event Filter + Event Consumer + Binding => WMI
- PowerShell/mofcomp.exe to setup
#### Hunting
##### SysInternal Autorun
##### Kansa PowerShell framework
##### PowerShell cmdlet Get-WmiObject
- Get-WMIObject -Namespace rootSubscription -Class __EventFilter
- Get-WMIObject -Namespace rootSubscription -Class __EventConsumer
- Get-WMIObject -Namespace rootSubscription -Class __FilterToConsumerBinding
#### Links
- [Monitoring Events](https://learn.microsoft.com/fr-fr/windows/win32/wmisdk/monitoring-events?redirectedfrom=MSDN)
- [There's nothing About WMI](https://www.youtube.com/watch?app=desktop&v=JCJl2uV8u1c)
- [Kansa & WMI Event Consumer](https://trustedsignal.blogspot.com/2014/05/kansa-collecting-wmi-event-consumer.html)
### More Advanced
- Local GPO
- MS Office Add-In
- BIOS flashing

## Credential theft
### Compromission tools
- Incognito
- MetaSploit
- Powershell
- Mimikatz
- fgdump
- gsecdump
- AceHash
- PWDumpX
- creddump
- WCE
- Cachedump
- Cain
- Kerberoasting
- ntdsutil
- VSSAdmin
- NTDSXtract
- secretsdump.py
- Bloodhound + Empire with Death Star
- GoFetch
### Hunting
#### EventLog 4624
| Admin action | Logon Type | Creds on target? | Notes |
|-|-|-|-|
| Console logon | 2 | Yes* | * Except when Credential Guard is enabled |
| Run as | 2 | Yes* | * Except when Credential Guard is enabled |
| Remote desktop | 10 | Yes* | * Except when Credential Guard is enabled |
| Net Use | 3 | No |Including /u: parameter|
| Powershell Remoting | 3 | No |Invoke-Command or Enter-PSSession|
| PsExec alternate creds | 3 + 2 | Yes | -u <username> -p <password>|
| PsExec w/o explicit creds | 3 | No | |
| Remote Scheduled Task | 4 | Yes | Password saved as LSA Secret|
| Runas service | 5 | Yes | (w/user account) - Password saved as LSA Secret |
| Remote registry | 3 | No | |
#### HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest with "UseLogonCredential" set to 1
### Info
#### Hashes
##### Defending
###### Stop remote interactive session with highly priviledged accounts
###### Ensure local admin account passwords are unique & not shared
###### Proper termination of RDP sessions
- Win8+:  force use of Restricted Admin
- Win10+: deploy Remote Credential Guard
###### Upgrade to Win10+
- Credential Guard
- WDigest & TsPkg creds no longer stored by default
- Domain protected users (mitiagte Pass The Hash attacks)
##### Links
- [Microsoft Security Advisory Update to Improve credentials Protection](https://support.microsoft.com/en-us/topic/microsoft-security-advisory-update-to-improve-credentials-protection-and-management-may-13-2014-93434251-04ac-b7f3-52aa-9f951c14b649)
- [Technet credentials protection & Mgmt](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn408190(v=ws.11)?redirectedfrom=MSDN)
- [Blocking remote use of local accounts](https://learn.microsoft.com/fr-fr/archive/blogs/secguide/blocking-remote-use-of-local-accounts)
- [Local Administrator Password solution](https://learn.microsoft.com/en-us/previous-versions/mt227395(v=msdn.10)?redirectedfrom=MSDN)
#### Token
##### Hunting / Defending
- Investigate users with SeImpersonate privilege
- Apply same defending actions as hashes password
- Use "Restricted Admin
- Use "Protected Users" security group
##### Links
- [Safeguarding Access Tokens](https://www.sans.org/blog/protecting-privileged-domain-accounts-safeguarding-access-tokens/)
- [Security implications of Windows Acess Token](https://dl.packetstormsecurity.net/papers/presentations/mwri_security-implications-of-windows-access-tokens_2008-04-14.pdf )
- [Monitoring delegation Token](https://www.sans.org/blog/monitoring-for-delegation-token-theft/)
- [Safeguarding Access Tokens](https://www.sans.org/blog/protecting-privileged-domain-accounts-safeguarding-access-tokens/)
- [Technet credentials protection & mgmt](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn408190(v=ws.11)?redirectedfrom=MSDN)
- ["Account is sensitive & cannot be delegated"](https://learn.microsoft.com/fr-fr/archive/blogs/poshchap/security-focus-analysing-account-is-sensitive-and-cannot-be-delegated-for-privileged-accounts)
- [Technet Protected Users Security Group](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn466518(v=ws.11)?redirectedfrom=MSDN)
#### Cached credentials
##### Info
- Cannot be used for Pass-The-Hash attack
- Cached credentials much be cracked
- Hashes are case-sensitive & salted
- Service accounts & computer accounts are also cached
##### Hunting/Defending
- SECURITY\Cache key
- \SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon CachedLogonsCount set to 4 (10 => 25 by default)
- Use of Protected Users security groups
- Enforce password length
##### Links
- [Cached logons & CachedLogonsCount](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/cached-domain-logon-information)
- [Domain Controller Security Technical Implementation Guide](https://www.stigviewer.com/stig/windows_server_20122012_r2_domain_controller/)
#### LSA secrets
##### Info
- Stored in the registry to allow services or tasks to be run with user privileges
- Can store application password like VPN or auto-logon creds
- The password is stored (not a hash)
##### Hunting/Defending
- SECURITY\Policy\Secrets
- SECURITY\Policy
- Do not employ services/schedule tasks requiring privileged accounts
- Reduce number of services requiring domain accounts
- Use (Group) Managed Service Accounts
##### Links
- [Avoid password mgmt with group managed services accounts](https://blog.stangroome.com/2013/07/28/avoid-password-management-with-group-managed-service-accounts/)
#### Tickets
##### Info
- Tickets are generated by Kerberos
- Tickets are chached in memory and are valid for 10 hours
- Can be used as Pass the Ticket
- DCs can provide ticket with no expiration (Golden ticket)
##### Kerberos Attacks detail / mitigation
| Attack Name | Detail | Mitigation |
|-|-|-|
| Pass the ticket | Steal ticket from memory and pass or import on other systems | Credential Guard; Remote Credential Guard |
| Overpass the Hash | Use NT hash to request a service ticket for the same account | Credential Guard; Protected Users Group; Disable RC4 authentication |
| Kerberoasting | Request service ticket for highly privileged service & crack NT hash | Long & complex service account passwords; Managed service account |
| Golden Ticket | Kerberos TGT for any account with no expiration. Survives full password reset | Protect domain admin accounts; change KRBTGT password regularly | 
| Silver Ticket | All-access pass for a single service or computer | Regular computer account password update |
| Skeleton Key | Patch LSASS on domain controller to add backdoor password that works for any domain account| Protect domain admin accounts; smart card usage for privileged account |
| DCSync | Use fake Domain Controller replication to retrieve hashes (and hash history) for any account without login to the DC | Protect domain admin; audit/limit accounts with replication rights |
##### Defending
- Change KRBTGT password regularly
- Audit service accounts for unusual activity
- Credential Guard
- Remote Credential Guard
- Long & complex passwords on service accounts
##### Links
- [Protecting Windows Networkds - Kerberos Attacks](https://dfirblog.wordpress.com/2015/12/13/protecting-windows-networks-kerberos-attacks/)
- [Protection from Kerberos Golden ticket](https://media.cert.europa.eu/static/WhitePapers/UPDATED%20-%20CERT-EU_Security_Whitepaper_2014-007_Kerberos_Golden_Ticket_Protection_v1_4.pdf)
- [Exploring the limitations of Remote Credential Guard](https://www.cyberark.com/resources/blog/no-more-pass-the-hash-exploring-the-limitations-of-remote-credential-guard)
- [Github Kerberoast](https://github.com/nidem/kerberoast)
- [Kerberos in the Crosshairs: Golden tickets, Silver iickets, MITM and more](https://www.sans.org/blog/kerberos-in-the-crosshairs-golden-tickets-silver-tickets-mitm-and-more/)
#### NTDS.DIT
##### Info
- Stored in %SystemRoot%\NTDS
- Location can be modified in HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters
##### Links
- [GitHub - Bloodhound](https://github.com/BloodHoundAD/BloodHound)
- [Automating the Empire with Death Star](https://byt3bl33d3r.github.io/automating-the-empire-with-the-death-star-getting-domain-admin-with-a-push-of-a-button.html)
-[GitHub - GoFetch](https://github.com/GoFetchAD/GoFetch)

## Program Execution
#### Prefetch
##### Info
- The original purpose is to improve system performence
- FileName like : <APPNAME>-<PATH_HASH>.pf
- Created after first execution of the app
- Logs all files and directories referenced by app
- Logs original path of execution
- Logs the total #of time app has been launched
- Logs the last time of execution
- From 128 files (win7 & before) to 1024 files (Win10 - Win11)
- Logs dates with ~10 seconds of delay

##### Hunting
###### Files in c:\Windows\Prefetch
###### HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters - EnablePrefetcher key
###### HKLM\SYSTEM\CurrentControlSet\Services\SysMain - Start key
##### Tool
- Zimmerman's tool: PECmd.exe
##### Links
- [Prefetch Hash Calculator](http://www.hexacorn.com/blog/2012/06/13/prefetch-hash-calculator-a-hash-lookup-table-xpvistaw7w2k3w2k8/)
- [Disabling Prefetch](https://learn.microsoft.com/en-us/previous-versions/windows/embedded/ms940847(v=winembedded.5)?redirectedfrom=MSDN)
- [Wht's new in Windows Application Execution](https://www.sans.org/blog/what-is-new-in-windows-application-execution/)
#### ShimCache
##### Info
- Original purpose: identify if an app needs to be shimmed for retro-compatibility
- Tracks Last modification date & file path of the executable file
- Good for proving application was moved, renamed or timestamped
- Most recent events are on top (last OS versions do not record execute time)
- New entries are only written on shutdown (or reboot)
- / ! \ Since Vista, the InsertFlag does not guarantee the application was executed
- May be helpful to provide clue of existence of the tool if an attacker removed it & the prefetch file
##### Hunting
- SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatCache\AppCompatCache (Win7+) - 1024 entries, InsertFlag gives some indication of execution but it is not definitive
- SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatibility\AppCompatCache (WinXP-) - 96 entries, (2003 - up to 512)
##### Links
- [Secrets of the Aplication Compatibility Database](https://web.archive.org/web/20191115211402/http://www.alex-ionescu.com/?p=39)
- [Understanding Shims](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-7/dd837644(v=ws.10)?redirectedfrom=MSDN)
- [Leveraging the application compatibility Cache in forensic investigation](https://web.archive.org/web/20190209113245/https://www.fireeye.com/content/dam/fireeye-www/services/freeware/shimcache-whitepaper.pdf)
##### Tools
- [Zimmerman's AppCompatCacheParser](https://github.com/EricZimmerman/AppCompatCacheParser)
- [Mandiant's ShimCacheParser](https://github.com/mandiant/ShimCacheParser) 
#### AmCache
##### Info
- Win7+ only
- Tracks installed applications, programs executed, drivers loaded & more
- Provides full path, file size, SHA1 hash, publisher metada & some timestamps of app & driver
- Hash available for app <= 31.4Mo (due to performance), but quite enougth for investigation purpose
- Entries can be due to automated file discovery or program installation.
=> does not always indicate a program execution
##### Hunting
- C:\Windows\AppCompat\Programs\Amcache.hve
- Can be used as an indication of executable & driver presence on the system
- "InventoryApplicationFile" provides some information on the application
We then can use the "ProgramId" to pivot to "InventoryApplication" to get more info
- "InventoryDriverBinary" provides info in drivers on the system
we can look for anomalies hashes, signature, metadata or modification time (matching suspicious activity timeframe)
##### Tools
- [Zimmerman's AmcacheParser](https://github.com/EricZimmerman/AmcacheParser)
- [appcompatprocessor.py](https://github.com/mbevilacqua/appcompatprocessor)
- [ShimCache & Amcache enterprise-wide hunting](https://www.youtube.com/watch?v=-0bYcD3_bBs)
##### Links

## Lateral Movement
### RDP - VNC - TeamViewer
#### Source Artifacts
##### EventLogs
- Security.evtx: 4648
- Microsoft-Windows-TerminalServices-RDPClient%4Operational.evtx: 1024 & 1102
##### Registry
- NTUSER\Siftware\Microsoft\Terminal Server Client\Server - Remote Desktop destinations are tracked per user
- ShimCache SYSTEM - mstsc.exe => Remote Desktop Client
- BAM/DAM SYSTEM - Last time executed
- BAM/DAM - mstsc.exe
- AmCache.hve - mstsc.exe => First Time executed
- UserAssist NTUSER.DAT - mstsc.exe => Last Time & #ofTime executed
- RecentApps NTUSER.DAT - mstsc.exe => Last Time & #ofTime executed
- RecentApps NTUSER.DAT - mstsc.exe => RecentItems => connection destinations & times
##### File System
- JumpLists C:\User\<USERNAME>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\{MSTCS-APPID}-automaticDestinations-ms => Destination & times
- Prefetch C:\Windows\Prefetch => mstsc.exe-<hash>.pf
- Bitmap Cache C:\Users\<USERNAME>\AppData\Local\Microsoft\Terminal Server Client\Cache - bcache##.bmc
- Bitmap Cache C:\Users\<USERNAME>\AppData\Local\Microsoft\Terminal Server Client\Cache - cache####.bmc
#### Destination Artifacts
##### EventLogs
- security.evtx - EventID 4624 logon type 10 => SourceIP & UserName
- security.evtx - EventID 4778/4779 => SourceIP & UserName
- Microsoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational.evtx - EventID 131 => Connection attempts (SourceIP)
- Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational.evtx - EventID 98 => Successful connection
- Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational.evtx - EventID 1149 => SourceIP & UserName (blank if using sticky key)
- Microsoft-Windows-TerminalServices-LocalSessionManager/Operational.evtx - EventID 21,22,25 => SourceIP & UserName 
- Microsoft-Windows-TerminalServices-LocalSessionManager/Operational.evtx - EventID 41 => UserName 
##### Registry
- ShimCache SYSTEM => rdpclip.exe & tstheme.exe
- Amcache.hve - rdpclip.exe & tstheme.exe => First Time executed
##### File System
- Prefetch C:\windows\Prefetch - rdpclip.exe-<HASH>.pf & tstheme.exe-<HASH>.pf
### Windows Admin Sharing
#### Source Artifacts
##### EventLogs
- security.evtx - EventID 4648 => DestinationIP & DestinationUserName, app
- Mcirosoft-Windows-SMBClient/Security.evtx EventID 31001 (Failed logon to destination) => Destination, UserName, reason why
##### Registry
- NTUSER\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2 - Remotely mapped shares
- Shellbags USRCLASS.DAT - Remote folder accessed inside an interactive session via explorer
- ShimCache SYSTEM - net.exe & net1.exe
- BAM/DAM NTUSER.DAT - net.exe & net1.exe => Last execution time
- AmCache.hve - net.exe & net1.exe => First execution time
##### File System
- Prefetch C:\windows\Prefetch - net.exe-<HASH>.pf & net1.exe-<HASH>.pf
#### Destination Artifacts
##### EventLogs
- security.evtx - EventID 4624 logon type 3 => SourceIP & UserName
- security.evtx - EventID 4772/4776 => SourceIP & UserName
- security.evtx - EventID 4768/4679 TGT Granted/Service Ticket Granted => SourceIP & UserName
- security.evtx - EventID 5140/5145 => Share Access/Auditing of shared files (noisy)
##### Registry
- None
##### File System
- File creation - Look for Modified Time before Creation Time
- File creation - Creation Time == Copy Time

### PsExec
#### Source Artifacts
##### EventLogs
- Security.evtx EventID 4648 => Current logged-on UserName, Altername UserName, DestinationIP, ProcesName
##### Registry
- NTUSER\Software\SysInternals\PsExec\EulaAccepted
- ShimCache SYSTEM - psexec.exe
- BAM/DAM NTUSER.DAT - psexec.exe => Last execution time
- AmCache.hve - psexec.exe => First execution time
##### File System
- Prefetch C:\windows\Prefetch - psexec.exe-<HASH>.pf
- File Creation psexec.exe => file downloaded & created on local host
##### Info - TIPS
- Log activity is more important in destination UNLESS the attacker uses the "runas" option (eventID 4648).
In that specific case, the source logs more stuff.
#### Destination Artifacts
##### EventLogs
- Security.evtx EventID 4624/4672 (logon type 3 (&2 if alternative user account used))=> -on UserName, SourceIP
- Security.evtx EventID 5140 => ADMIN$ share used by PsExec
- System.evtx EventID 7045 => Service installation
##### Registry
- SYSTEM\CurrentControlSet\Services\PSEXESVC - ('-r' option allows attacker to change service name)
- ShimCache SYSTEM - psexec.exe
- AmCache.hve - psexec.exe => First execution time
##### File System
- Prefetch C:\windows\Prefetch - psexesvc.exe-<HASH>.pf & evil.exe-<HASH>.pf
- File Creation psexesvc.exe => in ADMIN$ by default as well as evil.exe
- User profile structure created unless '-e' option used
##### Info - TIPS
- Multiple steps to remotly execute commands:
a. Authenticate to the destination
b. Name pipe set up between source & destination
c. ADMIN$ share mount on destination
d. PsExesvc & binaries copied to destination
e. Windows service started & cpoied files executed
- If script/binary created very close to the executoin of PSEXESVC.exe, the two are very likely related
- The Metasploit version of PsExec uses a random service name in exchange for Psexesvc, making
it easy to identify as evil. PsExec will create a user profile on the destination system by default. 
This presumes a profile doesn't already exist & that the attacker did not include the '-e' option. 
the creation time of this profile & it corresponding NTUSER.DAT registry data, can be another indicator of the PsExec activity
- RAM analysis can allows us to find named pipes. It may look like this:
\\<LOCALIP>\pipe\PSEXESVC-<sourcehostname>-<PID>-stdin
\\<LOCALIP>\pipe\PSEXESVC-<sourcehostname>-<PID>-stdout
\\<LOCALIP>\pipe\PSEXESVC-<sourcehostname>-<PID>-stderr
- If attacker can use '-r' option to change the name of PSEXESVC.
All artifacts still good but anyway
### Windows Remote Service
#### Source Artifacts
##### EventLogs
- None
##### Registry
- ShimCache SYSTEM - sc.exe
- AmCache.hve - sc.exe => First execution time
- BAM/DAM SYSTEM - sc.exe => Last execution time 
##### File System
- Prefetch C:\windows\Prefetch - sc.exe-<HASH>.pf & evil.exe-<HASH>.pf
#### Destination Artifacts
##### EventLogs
- security.evtx - EventID 4624 logon type 3 => SourceIP & UserName
- security.evtx - EventID 4697 => Service install, if enabled
- system.evtx - EventID 7034 => Service crashed unexpectedly
- system.evtx - EventID 7035 => Service sent a Start/Stop control
- system.evtx - EventID 7036 => Service Started/Stoped
- system.evtx - EventID 7040 => Service Start type changed (Boot, On Request, Disabled)
- system.evtx - EventID 7045 => Service was installed on the system
##### Registry
- SYSTEM\CurrentControlSet\Services => New service creation
- ShimCache SYSTEM - evil.exe => Records the existence of service executable unless implemented as a service DLL
- Amcache.hv - evil.exe => First execution time
##### File System
- File Creation - evil.exe or evil.dll creation on disk
- Prefetch C:\windows\Prefetch - evil.exe-<HASH>.pf
### Windows Remote ScheduledTask
#### Source Artifacts
##### EventLogs
- security.evtx EventID 4648  => DestinationIP, ProcessName & UserNames
##### Registry
- ShimCache SYSTEM - at.exe & schtasks.exe
- AmCache.hve - at.exe & schtasks.exe => First execution time
- BAM/DAM SYSTEM - at.exe & schtasks.exe => Last execution time 
##### File System
- Prefetch C:\windows\Prefetch - schtasks.exe-<HASH>.pf & at.exe-<HASH>.pf
#### Destination Artifacts
##### EventLogs
- security.evtx - EventID 4624 logon type 3 => SourceIP & UserName
- security.evtx - EventID 4672 => UserName, adminRights?, Requirement for default shares such as C$ or ADMIN$
- security.evtx - EventID 4698 => Scheduled Task created
- security.evtx - EventID 4702 => Scheduled Task updated
- security.evtx - EventID 4699 => Scheduled Task deleted
- security.evtx - EventID 4700/4701 => Scheduled Task Enabled/Disabled
- Microsoft-Windows-TaskScheduler/Operational.evtx EventID 106 => Scheduled Task created
- Microsoft-Windows-TaskScheduler/Operational.evtx EventID 140 => Scheduled Task updated
- Microsoft-Windows-TaskScheduler/Operational.evtx EventID 141 => Scheduled Task deleted
- Microsoft-Windows-TaskScheduler/Operational.evtx EventID 200/201 => Scheduled Task Executed/Completed
##### Registry
- SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks
- SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks\Tree
- ShimCache SYSTEM - evil.exe
- Amcache.hv - evil.exe => First execution time
##### File System
- File Creation - evil.exe creation on disk
- Job Files Creation in C:\Windows\Tasks
- XML Task Files Creation in C:\Windows\System32\Tasks
- Prefetch C:\windows\Prefetch - evil.exe-<HASH>.pf
### Windows Remote WMI
#### Source Artifacts
##### EventLogs
- security.evtx EventID 4648  => DestinationIP, ProcessName & UserNames
##### Registry
- ShimCache SYSTEM - wmic.exe
- AmCache.hve - wmic.exe => First execution time
- BAM/DAM SYSTEM - wmic.exe => Last execution time 
##### File System
- Prefetch C:\windows\Prefetch - wmic.exe-<HASH>.pf
#### Destination Artifacts
##### EventLogs
- security.evtx - EventID 4624 logon type 3 => SourceIP & UserName
- security.evtx - EventID 4672 => UserName, adminRights?
- Microsoft-Windows-WMI-Activity/Operational.evtx EventID 5857 => Time of wmiprvse execution & path to provider DLL
- Microsoft-Windows-WMI-Activity/Operational.evtx EventID 5860/5861 => Registration of temporaty/permanent event consumer
##### Registry
- ShimCache SYSTEM - scrcons.exe, mofcomp.exe, winprvse.exe, evil.exe
- Amcache.hve - scrcons.exe, mofcomp.exe, winprvse.exe, evil.exe => First execution time
##### File System
- File Creation - evil.exe & evil.mof creation on disk
- Prefetch C:\windows\Prefetch - evil.exe-<HASH>.pf, scrcons.exe-<HASH>.pf, mofcomp.exe-<HASH>.pf, winprvse.exe-<HASH>.pf
- Unauthorized changes to C:\windows\System32\wbem\Repository
### Windows Remote PowerShell
#### Source Artifacts
##### EventLogs
- security.evtx EventID 4648  => DestinationIP, ProcessName & UserNames
- Microsoft-Windows-WMI/Operational.evtx EventID 6 => Session created, DestionationIP, UserName
- Microsoft-Windows-WMI/Operational.evtx EventID 8,15,16,33 => Session closed, DestionationIP, UserName
- Microsoft-Windows-PowerShell/Operational.evtx EventID 40691/40692 => Local initiation of powershell & associated account
- Microsoft-Windows-PowerShell/Operational.evtx EventID 8193/8194 => Session created
- Microsoft-Windows-PowerShell/Operational.evtx EventID 8197 Connect => Session closed
##### Registry
- ShimCache SYSTEM - powershell.exe
- AmCache.hve - powershell.exe => First execution time
- BAM/DAM SYSTEM - powershell.exe => Last execution time 
##### File System
- Prefetch C:\windows\Prefetch - powershell.exe-<HASH>.pf => ps1 file will be tracked
- C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt => last 4096 commands launched by user
#### Destination Artifacts
##### EventLogs
- security.evtx - EventID 4624 logon type 3 => SourceIP & UserName
- security.evtx - EventID 4672 => UserName, adminRights?
- Microsoft-Windows-WinRM/Operational.evtx EventID 91 => Session created
- Microsoft-Windows-WinRM/Operational.evtx EventID 168 => Recodes the authenticating user
- Microsoft-Windows-PowerShell/Operational.evtx EventID 400/403 => Indicate start/end of remoting session
- Microsoft-Windows-PowerShell/Operational.evtx EventID 800 => Include partial script code
##### Registry
- ShimCache SYSTEM - wsmprovhost.exe, evil.exe
- Amcache.hve - wsmprovhost.exe, evil.exe => First execution time
- SOFTWARE\Microsoft\PowerShell\1\shellIds\Microsoft.PowerShell\ExecutionPolicy => May changed by attacker to reduce restriction
##### File System
- File Creation - evil.exe creation on disk
- File Creation - User profile directory (is 'Enter-PSSession' is used)
- Prefetch C:\windows\Prefetch - evil.exe-<HASH>.pf, wsmprovhost.exe-<HASH>.pf
### Other
#### Application Deployment Software
- Patch/application deployment tool are legitimate tools.
It implies that accounts & systems used in the patch deployment process must be heavily monitored
#### Vulnerability Exploitation
- Patch it !!!
### Links
- [RDP Bitmap Cache parser](https://github.com/ANSSI-FR/bmc-tools)

## (Native) EventLogs
|What|EvtxName|EventID|
|-|-|-|
|Logons|Security|4624, 4625, 4634, 4647<br>4648, 4672, 4720, 4726|
|Account Logon|Security|4768, 4769, 4771, 4776|
|RDP|Security<br>RDPCLient<br>RDPCoreTS<br>RemoteConnectionManager|4624, 4625, 4778, 4779<br>1024, 1102<br>98, 131<br>1149|
|Network Shares|Security|5140-5145|
|Sceduled Tasks|Security<br>Task Scheduler|4698<br>106, 140-141, 200-201|
|Installation|Application|1033, 1034, 11707, 11708, 11724|
|Services|System<br>Security|7034-7036, 7040, 7045<br>4697|
|Log Clearing|Security<br>System|1102<br>104|
|Malware Execution|Security<br>System<br>Application|4688<br>1001<br>1000-1002|
|Anti-Malware Log|Windows-Defender/Operational|1116-1119|
|Command Lines|Security<br>PowerShell/Operational|4688<br>4103-4104|
|WMI|WMI-Activity/Operational|5857-5861|

## Memory analysis
### Acquisition
#### Live System
- [WinPMEM](https://github.com/Velocidex/c-aff4/releases)
- [MagnetForensics RAM Capture](https://magnetforensics.com/free-tool-magnet-ram-capture)
- [Belkasoft Live RAM Capturer](https://forensic.belkasoft.com/en/ram-capturer)
- [F-Response](https://www.f-response.com)
#### Dead System
- Hibernation files - %SystemDrive%\hiberfil.sys - Contains a compressed RAM Image
- Pages & Swap files - %SystemDrive%\{pagefile.sys, swapfile.sys} - swapfile.sys => Win8+/2012+
- Kernel-Mode dump files - %SystemRoot%\MEMORY.DMP
- SYSTEM\CurrentControlSet\Control\CrashControl
- SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management
#### Links/Info
- MemProcFS can analyse these page files
-[PMEM Memory acquisition suite](https://github.com/Velocidex/c-aff4)
-[Varieties of Kernel-Mode Dump files](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/varieties-of-kernel-mode-dump-files)
-[Overview of memory dump file options for Windows](https://learn.microsoft.com/en-us/troubleshoot/windows-server/performance/memory-dump-file-options)
-[Win8/2012: the new swap file](https://learn.microsoft.com/en-us/archive/blogs/supportingwindows/windows-8-windows-server-2012-the-new-swap-file)
### Hibernation file tools
#### Decompress to raw
- Volatility v2's imagecopy plugin
- Volatility v3's layerwriter plugin
- hibr2bin.exe (not updated, so far, can't use on win10+)
- Arsenal's Hibernation Recon tool
#### Analyse natively 
- BulkExtractor
- Magnet AXIOM
- Volatility
- Passware
### Performing Memory Analysis
#### Identify Rogue Processes
##### Artifacts
- Image name: legitimate process, spelled correctly, matches system context
- Full path: appropriate path for tsystem executable, running from user/temp directory
- Parent process: is it the parent process expected, Orphan process
- Command line: executable matches image name, do arguments make sense
- Start time: was the process started at boot, processes started near time of known attack
- security ID: do the security identifiers make sense, why would a system process use a user account SID
##### Volatility commands
- windows.pslist: print all running processes within the EPROCESS doubly linked list
- windows.psscan: scan physical memory for EPROCESS pool allocation (can identify ended processes)
- windows.pstree: printprocess list as a tree showing parent relationships (using EPROCESS doubly linked list)
- Memory Baseliner: compare processes & loaded DLLs with a baseline image
#### Analyze process objects
##### Artifacts
- DLLs: Dynamic Linked Libraries (shared code)
- Handles - files: open files or I/O devices
- Handles - directories: lists of names used for access to kernel objects
- Handles - registry: access to a key within the windows registry
- Handles - mutexes/semaphore: control/limit access to an object
- Handles -  events: notifications that help threads communicate and organize
- Threads: smallest unit of execution; the workhorse of a process
- Memory section: shared memory areas used by a process
- Sockets: network port and connection information within a process
- SIDs: system processes should not be run with user's SID because they shouldn't start under user context
- Mutants: (or Mutexes) they control or limit access to a resource. Attackers sometimes use them to mark territory or avaoid reinfection
##### Volatility commands
- windows.dlllist: print list of DLLs for each process
- windows.cmdline: display commandline args for each process
- windows.getsids: extract the ownership SIDs for each process
- windows.handles: print list of open handles for each process
#### Review network artifacts
##### Artifacts
- Process communicating over port 80, 443 or 8080 that is not a browser
- Browser communicating over other port than 80, 443 or 8080
- Connection to unexlained internal/external IP addresses
- Web requests directly to an IP address rather than domain name
- RDP connections (port 3389), particularly if originating from odd/external IP
- DNS requests for unusual domain name
- Workstation to workstation connections
##### Volatility commands
- windows.netstat
- windows.netscan
#### Look for evidence of code injection
##### Artifacts
- DLL introduced using the Windows loarder (API)
- Identify unusual executable memory locations
- Uncover kernel and userland process inconsistencies
- VAD => (MappedPath, Base address)
- PEB => (Inload, InInit & InMem)
##### Volatility/MemProcFS/Live Analysis commands
- windows.ldrmodules
- windows.malfind
- windows.hollowfind (vol2)
- windows.ptemaldinf(vol3)
- findevil (MemProcFS)
- Moneta (Live Analysis)
- Hollow hunter (Live Analysis)
#### Audit drivers and rootkit detection
#### Dump suspicious processes & drivers
### Links/Info
- [Finding Kernel Global Variables](https://moyix.blogspot.com/2008/04/finding-kernel-global-variables-in.html)
- [Volatility CheatSheet](https://blog.onfvp.com/post/volatility-cheatsheet/)
- [Memory baseliner](https://github.com/csababarta/volatility_plugins/blob/master/baseline.py)
- [DarkCommat in memory](http://www.tekdefense.com/news/2013/12/23/analyzing-darkcomet-in-memory.html)
- [Well known Security Identifier](https://learn.microsoft.com/fr-fr/windows-server/identity/ad-ds/manage/understand-security-identifiers)
- [Access Token](https://learn.microsoft.com/fr-fr/windows/win32/secauthz/access-tokens?redirectedfrom=MSDN)
- [How Access Tokens work](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc783557(v=ws.10)?redirectedfrom=MSDN)
- CobaltStrike often use the same NamedPipe names: MSSE-####-server
- CobaltStrike often use the same NamedPipe names: msagent_##
- CobaltStrike often use the same NamedPipe names: status_##
- CobaltStrike often use the same NamedPipe names: postex_ssh_####
- CobaltStrike often use the same NamedPipe names: \\.\pipe\####### (from 7 to 10 #)
- CobaltStrike often use the same NamedPipe names: postex_####
- [MemProcFS tool](https://github.com/ufrisk/MemProcFS)
- [Harmony security Reflective DLL injection](https://www.dc414.org/wp-content/uploads/2011/01/242.pdf)
- [Analyzing Malware Hollow process](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/analyzing-malware-hollow-processes/)
- [Detecting deceptiv hollowing techniques](https://cysinfo.com/detecting-deceptive-hollowing-techniques/)
- [Threadmap](https://github.com/kslgroup/threadmap/blob/master/threadmap%20documentation.pdf)
- [PTEMalfind](https://insinuator.net/2021/12/release-of-pte-analysis-plugins-for-volatility-3/)
- [Gargoyle memory scanning evasion technique](https://github.com/JLospinoso/gargoyle)
- [Moneta Github](http://github.com/forrest-orr/moneta)
- [Hollow Hunter](https://github.com/hasherezade/hollows_hunter)
- [Harmony security Reflective DLL injection](https://www.dc414.org/wp-content/uploads/2011/01/242.pdf)
