This page outlines how to build Windows Mesos agent from source code.
## Prerequisites:
Microsoft Windows Server 1809 Core with Containers.

Docker container can be run with the following command:
`docker run -d -it --name mesos-build-manual mcr.microsoft.com/windows/servercore:ltsc2019-amd64 cmd`

All further steps need to be performed inside a container.

## Steps
1. Install Chocolatey package manager: 
`@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`
2. Install build tools (git, cmake, visualstudio 2017 community, visualstudio 2017 build tools and visual c++ build tools):
`choco install git cmake --installargs 'ADD_CMAKE_TO_PATH=System' visualstudio2017community visualstudio2017buildtools visualcpp-build-tools -y`
3. Install GnuWin32 patch.exe:
`mkdir c:\install && cd c:\install && wget.exe https://netix.dl.sourceforge.net/project/gnuwin32/patch/2.5.9-7/patch-2.5.9-7-setup.exe && .\patch-2.5.9-7-setup.exe /SILENT /SUPPRESSMSGBOXES`
4. Refresh environment variables (sometimes it need to be run twice):
`refreshenv`
5. Clone Mesos repo from git:
`mkdir c:\build && cd c:\build && git clone https://github.com/alekspv/mesos.git`
6. Configure using CMake for an out-of-tree build:
`mkdir c:\build\mesos\build && cd c:\build\mesos\build && cmake .. -G "Visual Studio 15 2017 Win64" -T "host=x64"`
7. Build Mesos:  
`cmake --build .`

Compilation process will take up to 1.5 hours on m5.xlarge ec2 node. After it will be finished, you can find compiled exe files in `src` subdirectory:

    c:\build\mesos\build>dir src | findstr /M /C:".exe" | findstr /M /C:"mesos"
    07/22/2019  10:05 AM        81,064,960 mesos-agent.exe
    07/22/2019  10:02 AM        31,820,800 mesos-containerizer.exe
    07/22/2019  10:02 AM        43,671,040 mesos-default-executor.exe
    07/22/2019  10:03 AM        44,639,232 mesos-docker-executor.exe
    07/22/2019  10:05 AM       103,521,792 mesos-execute.exe
    07/22/2019  10:03 AM        44,276,224 mesos-executor.exe
    07/22/2019  10:03 AM        37,594,624 mesos-fetcher.exe
    07/22/2019  10:06 AM        67,077,632 mesos-master.exe
    07/22/2019  10:06 AM        34,448,896 mesos-resolve.exe
    07/22/2019  10:04 AM           914,432 mesos-tcp-connect.exe
    07/22/2019  10:04 AM        31,708,672 mesos-usage.exe