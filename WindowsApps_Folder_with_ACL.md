
### WindowsApps Folder with ACL

>@DrewNaylor File Explorer always runs unelevated, Administrators also have access to C:\\Program Files\\WindowsApps yet you simply can't open it in File Explorer without breaking ACLs no matter how you try.

From above said it not correct

The Folder ACL: 
![Pasted image 20240605135146](https://github.com/mh57k/ODON/assets/37887919/a282fb41-809f-45a9-baf6-30713da17a1c)

![Pasted image 20240605135535](https://github.com/mh57k/ODON/assets/37887919/a7911f80-88b1-4e41-a75c-d53de8fd3661)

This ACE is only applied if the "WIN://SYSAPPID" system attribute exists.

mean that, if the process token has the _WIN://SYSAPPID_ attribute, it can access the Folder directly.


*Below Script can run as a normal user, to create a explorer process with token value*
```powershell

Install-Module -Name NtObjectManager
Import-Module -Name NtObjectManager


ls 'C:\Program Files\WindowsApps\'

$ps = Get-NtProcess -FilterScript {
  Use-NtObject($token = Get-NtToken -Process $_ -Access Query, Duplicate) {
     "WIN://SYSAPPID" -in $token.SecurityAttributes.Name -and -not $token.AppContainer
  }
}

# Verify The Token
$token = Get-NtToken -Process $ps[0] -Duplicate
Invoke-NtToken $token { 
   ls 'C:\Program Files\WindowsApps\' 
}


$ex = Get-NtProcess -Name 'explorer.exe'
$ex.Terminate(1)
$token = Get-NtToken -Process $ps[1] -Duplicate -TokenType Primary
$p = New-Win32Process "explorer.exe" -CreationFlags Suspended
Set-NtToken -Process $p -Token $token
Resume-NtProcess -Process $p

```

Reference:
[Tyranid's Lair: Working your way Around an ACL (tiraniddo.dev)](https://www.tiraniddo.dev/2024/06/working-your-way-around-acl.html)
