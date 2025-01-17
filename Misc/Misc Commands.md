
## Windows

[findstr examples](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/findstr#examples)



>- `Filter` parameter is implemented by provider. It is efficient because applies when retrieving the objects. Get-PSprovider commandlet shows providers that implement 'filter' parameter. For example, there are only two providers on my system:ActiveDirectory and FileSystem
>
>- `Include` parameter is implemented by Powershell. It only works in conjunction with `Recurse` parameter




#### Force color terminal
```powershell
REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1 
```


#### List number of files a shared drive & its subdirectories have
```powershell
# CMD
dir n: /a-d /s /b | find /c ":\"

# PS
N:
PS N:\> (Get-ChildItem -File -Recurse | Measure-Object).Count
```
	- /a-d - Attributes ; Not(-) Directories
	- /s - Displays files in a specified directory and all subdirectories
	- /b - Uses bare format (no heading information or summary)



## Linux
#### Write terminal color output to file
```bash
script -q -c "feroxbuster --url http://aero.com --filter-status 404" feroxbuster-80.log
```

