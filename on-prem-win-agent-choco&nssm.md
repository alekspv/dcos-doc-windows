This page outlines how to install and connect Windows private agent to master node for production work (with Chocolatey & NSSM).
# Prerequisites:
DC/OS Cluster, Microsoft Windows Server 1809 Core with Containers

## Turn Off Windows Firewall Using PowerShell
Microsoft PowerShell will be used as default shell. In order to initiate PowerShell screen, please, launch the PowerShell with the following command line:
```powershell
PowerShell.exe
```
Turn Off Windows Firewall
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```
Disable Windows Defender Security Center
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```
## Installing Chocolatey with PowerShell.exe

Run the following command

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

## Download and install Microsoft Visual C++ 2015 with Chocolatey

Mesos agent for Windows requires Visual C++ 2015 Redistributable package. Download and install the [Visual C++ 2015 Redistributable package](https://www.microsoft.com/en-us/download/details.aspx?id=48145)

```powershell
choco install vcredist140
```
After installation is complete, then OS requires the system restart
```powershell
shutdown -r -t 0
```
## Download and install NSSM with Chocolatey

The NSSM is a computer program that wraps arbitrary programs thus enabling them to be installed and run as Windows Services or Unix daemons, programs that run in the background, rather than under the direct control of a user.

```powershell
choco install nssm
```
## Creating a local folder 

```powershell
md C:\dcos; md C:\dcos\images; md C:\dcos\work; md C:\dcos\mesos-logs
```

## Download and extract Mesos-binaries

Load the assembly System.IO.Compression.FileSystem  and use ExtractToDirectory from System.IO.Compression.ZipFile PowerShellowershell modules

```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
function Unzip {
    param([string]$zipfile, [string]$outpath)
    [System.IO.Compression.ZipFile]::ExtractToDirectory($zipfile, $outpath)
}
```


```powershell
cd c:\dcos\

Invoke-WebRequest -OutFile mesos-binaries.zip dcos-win.westus2.cloudapp.azure.com/artifacts/dcos-mesos-build/1334/mesos/binaries/mesos-binaries.zip

Unzip "C:\dcos\mesos-binaries.zip" "C:\dcos\mesos-binaries"
```

## Create a service

`mesos-agent.exe` requires following parameters

| parameter  | description |
| ------------- | ------------- |
|--master | `host:port` |
|  | `zk://host1:port1,host2:port2,.../path`  |
|  |`zk://username:password@host1:port1,host2:port2,.../path`|
|  --appc_store_dir |Directory the appc provisioner will store images in. (default: C:\Users\ADMINI~1\AppData\Local\Temp\2\mesos\store\appc) | 
|--work_dir|Path of the agent work directory. This is where executor sandboxes will be placed, as well as the agent's checkpointed state in case of failover. Note that locations like `/tmp` which are cleaned automatically are not suitable for the work directory when running in production, since long-running agents could lose data when cleanup occurs. (Example: `/var/lib/mesos/agent`) (default: )|
|--runtime_dir|Path of the agent runtime directory. This is where runtime data is stored by an agent that it needs to persist across crashes (but not across reboots). This directory will be cleared on reboot. (Example: `/var/run/mesos`) (default: C:\ProgramData\mesos\runtime)|
| --isolation|Isolation mechanisms to use, e.g., `posix/cpu,posix/mem` (or `windows/cpu,windows/mem` if you are on Windows), or `cgroups/cpu,cgroups/mem`, or `network/port_mapping`  (configure with flag: `--with-network-isolator` to enable), or `gpu/nvidia` for nvidia specific gpu isolation, or load an alternate isolator module using the `--modules` flag. if `cgroups/all` is specified, any other cgroups related isolation options (e.g., `cgroups/cpu`) will be ignored, and all the local enabled cgroups subsystems on the agent host will be automatically loaded by the cgroups isolator. Note that this flag is only relevant for the Mesos Containerizer. (default: windows/cpu,windows/mem)|
|--containerizers|Comma-separated list of containerizer implementations  to compose in order to provide containerization. Available options are `mesos` and `docker` (on Linux). The order the containerizers are specified is the order they are tried. (default: mesos)|
|--launcher_dir|Directory path of Mesos binaries. Mesos looks for the fetcher, containerizer, and executor binary files under this directory. (default: WARNINGDONOTUSEME)|
|--log_dir|Location to put log files.  By default, nothing is written to disk. Does not affect logging to stderr. If specified, the log file will appear in the Mesos WebUI.  NOTE: 3rd party log messages (e.g. ZooKeeper) are  only written to stderr!|
|--logging_level|Log message at or above this level.   Possible values: `INFO`, `WARNING`, `ERROR`. If `--quiet` is specified, this will only affect the logs written to `--log_dir`, if specified. (default: INFO)|
|--attributes|Attributes of the agent machine, in the form: `rack:2` or `rack:2;u:1`|
|--ip|IP address to listen on. This cannot be used in conjunction with `--ip_discovery_command`.|
|--hostname|The hostname the agent should report. If left unset, the hostname is resolved from the IP address that the agent advertises; unless the user explicitly prevents that, using `--no-hostname_lookup`, in which case the IP itself is used|

*See `mesos-agent --help` for a list of more parameters*

Run in console (change values **master-local-ip** and **your-local-ip**)

```powershell
nssm install
```

**Path:** C:\dcos\mesos-binaries\mesos-agent.exe

**Startup directory:** C:\dcos\mesos-binaries


**Arguments:** --master=<master-local-ip>:5050  --appc_store_dir=\\?\C:\dcos\images\ --work_dir=\\?\C:\dcos\work\ --runtime_dir=\\?\C:\dcos\work\ --isolation="windows/cpu,windows/mem,filesystem/windows" --containerizers="mesos,docker" --launcher_dir=\\?\C:\dcos\mesos-binaries\ --log_dir=C:\dcos\mesos-logs --attributes="os:windows" --ip=<your-local-ip> --hostname=<your-local-ip>

press **Install service**

***set dependence***

in concole

```powershell
sc config mesos-agent depend= docker
```


Start mesos-agent service
```powershell
sc start mesos-agent
```
