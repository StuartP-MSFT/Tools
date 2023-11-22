# Tools

#iDNA CheckList & Recommended Practices 

Please read the following instructions carefully.  Any deviation from the steps below could result in delayed analysis. 

[[_TOC_]]


## Before 

1. Limit activity from other users as much as possible to help eliminate noise and reduce the size of trace. Ensure that you can force the repro to a single specific server
1. Ensure you have about 20GB free disk space 
1. IIS Reset – (only if tracing w3wp.exe) Flush everything from memory 
1. Reproduce the problem one time – This gets everything loaded back into memory and prevents the iDNA trace from capturing startup overhead activity in the background. 
1. Use Task Manager to determine the process ID (PID).  Using the Details tab with the “PID” and “Command line” columns is a relatively easy way to determine the PID. 
 
    If you have to debug a W3WP process, you should use the following command to get the correct Process: 
    ```C:\windows\system32\inetsrv\appcmd.exe list wp ```
 
1. Prepare, but do not execute, the command prompt for the trace.   
```C:\TTT_x64_x64_External\x64>TTTracer.exe -dumpfull –attach [PID from step 5] ```

   Note: Ensure that you're in the x64 folder as all SharePoint processes are 64bit. 

1. Start a new ULS log file 
```PS C:\Users\Administrator.CONTOSO> New-SPLogFile```
 
## During 
1. Execute the TTTracer command from **Before: Step 6** and wait for the tracing window

    Note: You may have to wait up to 5 minutes, but eventually a small window will appear that has a checkbox for **Tracing On**. Leave this window active.


1. Reproduce the problem in the fewest possible steps 
1. Wait 3-5 seconds then uncheck **Tracing On** in the window that appeared in **During: Step 1**. 
    The command prompt will then show the output location of the iDNA trace: 
``` 
C:\TTT_x64_x64_External\x64>TTTracer.exe -dumpfull -attach 400 
Microsoft (R) TTTracer 10.00.10011 (Feb 10 2015 13:49:47) (oobla) 
Copyright (C) Microsoft Corporation. All rights reserved. 
 
w3wp.exe(x64) (400): Tracing stopped after 31171ms 
  Full trace dumped to C:\TTT_x64_x64_External\x64\w3wp01.run 
```
 
## After 
1. Start a new ULS log file 

   ```PS C:\Users\Administrator.CONTOSO> New-SPLogFile ```
1. Revert to normal ULS Logging if you increased any logging (increasing logging not recommended for IDNA)

   ```PS C:\Users\Administrator.CONTOSO> Clear-SPLogLevel ```

1. Review the ULS logs for the error we are trying to capture.  Does the ULS process ID match the process ID in **Before: Step 5**? 

   NOTE: ULS displays process IDs in hex, while task manager uses decimal values.] 

1. Combine the .RUN, .OUT, ULS  files into a single archive (you may only have 2 of these files, this is ok).  The built in Windows zip function usually works well.  But for extremely large files (5GB or greater), you may need to use WinRAR or 7-Zip.  
 
    Please verify the archive can be extracted before uploading. 
 
1. Upload the archive to the workspace provided by the Microsoft support engineer. 

## Identifying the Process ID (PID)
There are multiple ways to identify the process ID for the task you need to trace. 

### Task Manager
To Identify the Process ID (PID) of the task you are interested in Tracing. If you are debugging an app pool a real simple way to identify the correct on is
1. Start TASKMGR, click on the Details tab and right click on the column headers. 
1. Click on Select Column
1. Check Command Line (it's usually toward the bottom)
1. Close that dialog, Sort by the process name and find the correct W3WP process based on the SharePoint application name. 

In this example the "SharePoint - 8080" Application is on PID 7348. To trace this we would issue the following cmd:

```TTTRacer -DumpFull -attach 7348```

### AppCmd
Alternatively, and perhaps easier, you can use APPCMD from the C:\windows\system32\inetsrv folder:

```
C:\Windows\System32\inetsrv>appcmd list wp
WP "5560" (applicationPool:SharePoint - 9999 (5.4.2023 7.14.41 PM))
WP "3652" (applicationPool:SecurityTokenServiceApplicationPool)
WP "6052" (applicationPool:4abe8530d12d4b70ac089ff4c83aa21c)
WP "7348" (applicationPool:SharePoint - 8080)
```

Here we can see that the current process ID for application pool "SharePoint - 8080" is 7348. 
