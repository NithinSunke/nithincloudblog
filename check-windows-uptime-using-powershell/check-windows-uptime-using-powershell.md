---
title: "Check Windows Uptime Using PowerShell"
author: "Sunke Nithin"
date: 2025-08-16
categories: ["BASH SCRIPTS"]
tags: []
source: https://nithinsunke.wordpress.com/2025/08/16/check-windows-uptime-using-powershell/
---

# Check Windows Uptime Using PowerShell

When managing Windows systems, one of the most common things administrators want to know is **how long the system has been running since the last reboot**. This is called *uptime*. Uptime monitoring is especially important for servers where stability, troubleshooting, and patch compliance depend on how long the machine has been online.

While Windows provides basic uptime info in **Task Manager**, **PowerShell** gives us a much more flexible and scriptable way to check uptime.

#### Using CIM/WMI

PowerShell can query the **Win32_OperatingSystem** class to get the **last boot time** of your system.

``` wp-block-code
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime
```

#### Full Script

``` wp-block-code
# uptime.ps1 - Display system uptime in days, hours, minutes, seconds
$lastBoot = (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
$uptime = (Get-Date) - $lastBoot

Write-Output "System Last Boot: $lastBoot"
Write-Output ("System Uptime   : {0} Days {1} Hours {2} Minutes {3} Seconds" -f `
    $uptime.Days, $uptime.Hours, $uptime.Minutes, $uptime.Seconds)
```

Run it with powershell:

``` wp-block-code
.\uptime.ps1
```

Example Output:

``` wp-block-code
System Last Boot: 8/10/2025 2:45:12 PM
System Uptime   : 6 Days 21 Hours 11 Minutes 15 Seconds
```

## Conclusion

Checking Windows uptime with PowerShell is quick, flexible, and script-friendly. Whether you want a simple one-liner, a detailed uptime report, or an automated uptime logger, PowerShell has you covered.
