# Windows Fundamentals

## Introduction to Windows

many versions of windows are now deemed legacy and are no longer supported   
orgs often find themselves running older OS to support critical apps due to operational or budget concerns   
a pen tester will need to understand the differences between versions and the various misconfigs and vulnerabilities in each one 

major windows OS and version numbers: 

![](Images/Pasted%20image%2020240416191535.png)

we can use `Get-WmiObject` cmdlet to find info about the OS   
can be used to get instances of WMI classes or info about available WMI classes   
the `win32_OperatingSystem` class shows info like version and build number: 

`Get-WmiObject -Class win32_OperatingSystem | select Version,BuildNumber`

![](Images/Pasted%20image%2020240417160145.png)

from this we can see that the build number is 19041 and the windows NT version is Windows 10

some other useful classes that can be used with `Get-WmiObject` are:   
- `Win32_Process` to get a process listing 
- `Win32_Service` to get a listing of services 
- `Win32_Bios` to get BIOS info 
- `ComputerName` parameter to get info about remote computers 

`Get-WmiObject` can be used to start and stop services on local and remote pcs 

### Accessing windows 

local access = most common way of accessing a computer; physical access to device   
input through keyboard or trackpad/mouse   
output coming from the screen  

remote access = accessing pc over a network   
local access is needed before someone can access another pc remotely   
industries like MSPs and MSSPs are primarily dependent on managing their client's pc remotely   

some of the most common remote access tech: 
- virtual private network VPN 
- secure shell SSH 
- file transfer protocol FTP 
- virtual network computing VNC 
- windows remote management (or powershell remoting) WinRM
- remote desktop protocol RDP 

### Remote desktop protocol 

rdp uses a client/server architecture where client-side app is used to specify a computer's target IP address or hostname over a network where RDP is enabled   
target pc with RDP enabled is considered the server   
listens by default on logical port 3389    

IP address is used as a logical identifier for a pc on a network  
logical port is an identifier assigned to an application   

request reaches destination computer via its IP address then it is directed to an app hosted on the computer based on the port specified 

simply, every pc has an IP address assigned to communicate over a network and apps hosted on target computers listen on specific logical ports 

if we are connecting to a windows target from windows we can use the built in RDP client `Remote Desktop Connection` (mstsc.exe): 

![](Images/Pasted%20image%2020240417162958.png)

for this to work, remote access must be allowed on the target windows system  
it is not enabled by default   

note that RDP also allows us to save connection profiles, which is a common habit among IT admins because it makes connections to remote systems easier   
we can benefit from this by looking for saved `Remote Desktop Files` (.rdp) 

there are other tools like Remote Desktop clients 

from a linux based attack host we can use a tool called `xfreerdp` to remotely access windows targets: 

```shell
xfreerdp /v:<targetIp> /u:htb-student /p:Password
```

![](Images/Pasted%20image%2020240417163245.png)

there are other tools like remmina and rdesktop

## Operating Systems Structure 

in windows the root directory is `<drive letter>:\` and it is commonly in the `C` drive   
also known as the boot partition, and is where the OS is installed 

there are other physical and virtual drives that are assigned other letters like `Data (E:)` 

the directory structure of the boot partition is as follows: 

![](Images/Pasted%20image%2020240417171500.png)

we can explore directories using the `dir` command: 

![](Images/Pasted%20image%2020240417172820.png)

![](Images/Pasted%20image%2020240417172921.png)

the `tree` utility will graphically display the directory structure of a path or disk: 

![](Images/Pasted%20image%2020240417173912.png)

you can alternatively view all of this info one page at a time with: 

```
tree c:\ /f | more
```

can use dir and cat to find flag file: 

![](Images/Pasted%20image%2020240417174154.png)

## File System 

there are 5 types of windows file systems:   
- FAT12 
- FAT16
- FAT32
- NTFS 
- exFAT 

FAT 12 and 16 are no longer used on modern windows OS   

FAT32 (file allocation table) is widely used across many types of storage devices like USB and SD cards but can also be used to format hard drives   
32 = 32 bits of data for identifying data clusters on a storage device 

pros of FAT32:   
- device compatibility - pcs, digital cameras, gaming consoles, smartphones, tablets, etc. 
- OS cross-compatibility - works on all windows OS from 95 and is also supported by mac and linux 

cons of FAT32:  
- can only be used with files that are less than 4gb 
- no build in data protection or file compression 
- must use 3rd part tools for encryption 

NTFS (new technology file system) is the default windows file system since windows NT 3.1   
makes up for FAT32 shortcomings and has better support for metadata and better performance 

pros of NTFS:   
- reliable and can restore consistency of the file system in the even of system failure or power loss 
- provides security by setting granular permissions on both files and folders 
- supports very large-sized partitions 
- has journaling built in meaning that file modifications are logged 

cons of NTFS: 
- most mobile devices do not support NTFS natively 
- older media devices like TVs and digital cameras do not offer support for NTFS storage devices 

some of the key permission types of NTFS are: 

![](Images/Pasted%20image%2020240417180620.png)

files and folders inherit the NTFS permissions of their parent folder for ease of admin   
if permissions dont need to be set explicitly, an admin can disable permissions inheritance 

### Integrity control access control list (icacls)

NTFS permissions on files and folders can be managed through the file explorer GUI under the security tab (properties)
for a finer level of granularity over NTFS we can use the `icacls` utility 

we can list out the NTFS permissions of a directory by running `icacls` from within the working directory or `icacls C:\Windows` for a directory we are not in: 

![](Images/Pasted%20image%2020240417182121.png)

the resource access level is listed after each user in the output  
possible inheritance settings are: 
- `(CI)`: container inherit 
- `(OI)`: object inherit 
- `(IO)`: inherit only 
- `(NP)`: do not propagate inherit 
- `(I)`: permission inherited from parent container 

basic access permissions are:   
- `F`: full access
- `D`: delete access
- `N`: no access
- `M`: modify access
- `RX`: read and execute access
- `R`: read-only access
- `W`: write-only access 

we can add and remove permissions via the command line with `icacls`   

here we are executing `icacls` in the context of a local admin account: 

![](Images/Pasted%20image%2020240417182626.png)

our example user "joe" doesn't have any permissions 

using `icacls c:\users /grant joe:f` we can grant joe full control over the directory, but since `(oi)` and `(ci)` were not included the user joe will only have rights over the `c:\users` folder but not over the user subdirectories and files contained within them: 

![](Images/Pasted%20image%2020240417182826.png)

these permissions could be revoked using `icacls c:\users /remove joe` 

`icacls` is very powerful and can give users specific permissions over a file or folder, explicitly deny access, enable or disable inheritance permissions, and change directory/file ownership 

we can see that the user "bob.smith" has full access control over the users directory: 

![](Images/Pasted%20image%2020240417183047.png)

