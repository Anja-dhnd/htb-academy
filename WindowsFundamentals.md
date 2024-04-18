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

