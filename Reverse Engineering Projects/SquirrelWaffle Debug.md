Malware Sample: https://malshare.com/sample.php?action=detail&hash=6095f96dd5eca96a3fb9338eec4ab574921c0febb36f6a6db60aae1aeb9ffcab

**Introduction**
According to Sophos, Squirrelwaffle is a malware loader that is distributed as a malicious Office document in spam campaigns. It provides attackers with an initial foothold in a victim’s environment and a channel to deliver and infect systems with other malware. When a recipient opens a Squirrelwaffle-infected document and enables macros, a visual basic script typically downloads and executes malicious files and scripts, giving further control of the computer to an attacker. Squirrelwaffle operators also use DocuSign to try and trick the user into enabling macros in Office documents.

**Debugging**
Clicking the malware sample link will bring you to this page. You can make an account and download the sample here:
![[Pasted image 20250330181955.png]]

Once downloaded the filename will need to be changed to .dll so that IDA will have an easier time reading the file. 

Upon opening IDA click new > select dll file 

![[Pasted image 20250330182530.png]]


Click Exports 

In exports we see two loaders: 
![[Pasted image 20250330182719.png]]

ldr is set as ordinal 1 which is where the config decryption occurs for this dll. 

DLLEntryPoint does not have anything interesting stored within it at this time and if we attempted to debug this we wouldn't be able to debug the code within the loader. 

Loading the export into x32dbg due to the dll being 32bit. 
![[Pasted image 20250330183142.png]]

For further testing a PE file is created by x32dbg so that it may load our malicious DLL for analysis. 
![[Pasted image 20250330183331.png]]

Bottom left ^^^

Right now we want to break the entry point of that DLL so that we may access the code behind the loader itself.

In options you'll want to enable User DLL Entry so that we can set the breakpoint between the two exports. From there we will run the program until we see the breakpoint occur. 
![[Pasted image 20250330184358.png]]

Breakpoint:
![[Pasted image 20250330184704.png]]
![[Pasted image 20250330184554.png]]

Right now the breakpoint is set to the DLLEntryPoint. We need to change the EIP to match the loader (ldr) and that will call the preferred export. 
![[Pasted image 20250330204308.png]]
![[Pasted image 20250330204345.png]]

Now we need to set a new breakpoint at the decryption portion here and then take a look at the return value of eax. 
![[Pasted image 20250330204800.png]]

After this we run the debugger and we right click at the bottom and follow DWORD in current dump which will reveal the blocklist IP addresses of the DLL config. 
![[Pasted image 20250330205221.png]]

Resources:
https://malpedia.caad.fkie.fraunhofer.de/details/win.squirrelwaffle
https://blog.talosintelligence.com/squirrelwaffle-emerges/
https://www.trendmicro.com/en_us/research/21/k/Squirrelwaffle-Exploits-ProxyShell-and-ProxyLogon-to-Hijack-Email-Chains.html
https://any.run/malware-trends/squirrelwaffle/
https://www.virustotal.com/gui/file/6095f96dd5eca96a3fb9338eec4ab574921c0febb36f6a6db60aae1aeb9ffcab
https://github.com/mandiant/flare-vm