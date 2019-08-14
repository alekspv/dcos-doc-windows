## DC/OS Distributed Diagnostics Tool & Aggregation Service (on-prem)
dcos-diagnostics is a monitoring agent which exposes a HTTP API for querying from the /system/health/v1 DC/OS api. dcos-diagnostics puller collects the data from agents and represents individual node health for things like system resources as well as DC/OS-specific services.

This page outlines how to install diagnostic service on Windows private agent for production work (with NSSM).

## Installing Chocolatey with PowerShell.exe

Run the following command

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```


## Download and install NSSM with Chocolatey

The NSSM is a computer program that wraps arbitrary programs thus enabling them to be installed and run as Windows Services or Unix daemons, programs that run in the background, rather than under the direct control of a user.

```powershell
choco install nssm
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

Invoke-WebRequest -OutFile dcos-diagnostics.zip https://dcos-win.s3.amazonaws.com/dcos-diagnostics.zip

Unzip "C:\dcos\dcos-diagnostics.zip" "C:\dcos"
```

## Create a service

Run dcos-diagnostics once, on a DC/OS host to check systemd units:

```
dcos-diagnostics --diag
```

Get verbose log output:

```
dcos-diagnostics --diag --verbose
```

Run the dcos-diagnostics aggregation service to query all cluster hosts for health state:

```
dcos-diagnostics daemon --pull
```

Start the dcos-diagnostics health API endpoint:

```
dcos-diagnostics daemon
```

### dcos-diagnostics daemon options

<pre>
--agent-port int
    Use TCP port to connect to agents. (default 1050)

--ca-cert string
    Use certificate authority.

--command-exec-timeout int
    Set command executing timeout (default 120)

--diag
    Get diagnostics output once on the CLI. Does not expose API.

--diagnostics-bundle-dir string
    Set a path to store diagnostic bundles (default "/var/run/dcos/dcos-diagnostics/diagnostic_bundles")

--diagnostics-job-timeout int
    Set a global diagnostics job timeout (default 720)

--diagnostics-units-since string
    Collect systemd units logs since (default "24 hours ago")

--diagnostics-url-timeout int
    Set a local timeout for every single GET request to a log endpoint (default 2)

--endpoint-config string
    Use endpoints_config.json (default "/opt/mesosphere/endpoints_config.json")

--exhibitor-ip string
    Use Exhibitor IP address to discover master nodes. (default "http://127.0.0.1:8181/exhibitor/v1/cluster/status")

--force-tls
    Use HTTPS to do all requests.

--health-update-interval int
    Set update health interval in seconds. (default 60)

--master-port int
    Use TCP port to connect to masters. (default 1050)

--port int
    Web server TCP port. (default 1050)

--pull
    Try to pull checks from DC/OS hosts.

--pull-interval int
    Set pull interval in seconds. (default 60)

--pull-timeout int
    Set pull timeout. (default 3)

--verbose
    Use verbose debug output.

--version
    Print version.
</pre>



**Run in PowerShell** 

Run the following command

```powershell
$env:DCOS_VERSION=<dcos-version>
```


```powershell
nssm install
```

**Path:** C:\dcos\dcos-diagnostics.exe

**Startup directory:** C:\dcos\


**Arguments:**  --config dcos-diagnostics-config.json --role agent daemon

press **Install service**

Start mesos-agent service
```powershell
sc start dcos-diagnostics
```
