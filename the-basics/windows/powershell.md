# PowerShell

PowerShell is Windows new shell. It comes by default from Windows 7. But can be downloaded and installed in earlier versions.

* PowerShell provides access to almost everything an attacker might want.
* It is based on the .NET framework.
* It is basically bash for windows
* The commands are case-insensitive

## Basics

So a command in PowerShell is called **cmdlet**. The cmdlets are created using a verb and a noun. Like `Get-Command`, Get is a verb and Command is a noun. Other verbs can be: remove, set, disable, install, etc.

To get help on how to use a **cmdlet** while in PowerShell, the man-page, you do:

```text
Get-Help    <cmdlet    name    |    topic    name>
```

Example

```text
get-help echo
get-help get-command
```

**Powershell Version and Build**

```text
$PSVersionTable
```

### Fundamentals

With get-member you can list all the properties and methods of the object that the command returns.

```text
Get-Member
For example:
Get-Command | Get-Member
Get-Process | Get-Membeobject
```

#### Variables

```text
$testVar = "blabla"
```

**File Download**

```text
# PowerShell One Liner, works on PowerShell 2.0+ 
    (New-Object Net.WebClient).DownloadFile('http://<uri>','c:\windows\temp\<file_name>')

# PowerShell Invoke-WebRequest, requires PowerShell 3.0+ 
    Invoke-WebRequest -Uri <uri> -OutFile <file_name>
```

**Searching through results**

```text
# Select-String (Grep alternative)
Get-Command | Select-String -Pattern "^get-"
```

**General PS cmdlets to interrogate objects**

Additional information about Powershell cmdlets can be found on [Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/?view=powershell-6)

```text
# Dump of useful PowerShell cmdlets for objects
    Select-Object
    Measure-Object
```

### Working with filesystem

**List all files in current directory**

```text
# List directory contents
    Get-ChildItem 

# List files in directory
    Get-ChildItem -File
    
# List Directories in Directory
    Get-ChildItem -Directory

# List File Attributes
    Get-ChildItem -Directory | Select-Object FullName, Attributes

# List Hidden Files 
    Get-ChildItem -Force 

# Recursively list files and folders
    Get-ChildItem | Measure-Object
```

### Working with files

```text
# Read conetents of a file 
    Get-Content -Path <file name / location>

# Count lines of a file
    Get-Content -Path <file name / location> 
    
# Select specific line in a file 
    (Get-Content -Path <file name / location>)[10] #prints line 11. 
  or
    Get-Content -Path <file name / location> | Select-Object -Index 10 #prints line 11.
```

### Services

Series of cmdlets that allow for the interaction with Services include: list, stop, start, and restart. 

```text
# List Services 
    Get-Service

# Stop Services 
    Stop-Service -Name <Service Name>

# Start Service 
    Start-Service -Name <Service Name> 

# Restart Service 
    Restart-Service -Name <Service Name> 
```

### Network related stuff

Domain information. I believe the following cmdlets require [RSAT ](https://technet.microsoft.com/en-us/library/gg413289.aspx)to be installed and enabled before use. 

```text
Get-ADDomain
Get-AdDomainController
Get-AdComputer
To see a list of all properties do this
get-adcomputer ComputerName -prop *

Get AD Users
Get-ADUser -f {Name -eq 'Karl, Martinez'} -properties *

Get all AD Groups
Get-ADGroup -filter *
```

