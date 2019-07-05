##  How to configure Marathon-LB for windows node
### 1. Make sure that Windows MESOS agent was ran with next parameters:

- `--attributes="os:windows"`	- will be used for constraint rules on DC-OS jobs like ("constraints": [["os","LIKE","windows"]])
- `--hostname=$env:HostIP`	- this value uses as a node name in Mesosphere end resolving for delegation jogs as well. Better to use privet node IP.

### 2. Installing Marathon-LB service on DC/OS
https://docs.mesosphere.com/services/marathon-lb/1.13/mlb-install/
##### 2.1 With DC-OS web interface:
Go to tab `Catalog`-> `search 'marathon-lb'` -> `Review & run`

##### 2.2 With `dcos` util:
```bash
dcos package install marathon-lb –yes
```

### 3. Preparing service configuration to run on windows node:
###### dotnet-website.json
```json
{
    "id": "dotnet2",
    "cmd": null,
    "cpus": 1,
    "mem": 1024,
    "disk": 0,
    "instances": 5,
    "constraints": [
      [
        "os",
        "LIKE",
        "windows"
      ]
    ],
    "acceptedResourceRoles": [
      "*"
    ],
    "container": {
      "type": "DOCKER",
      "docker": {
        "forcePullImage": false,
        "image": "mcr.microsoft.com/dotnet/core/samples:aspnetapp",
        "parameters": [],
        "privileged": false
      },
      "volumes": [],
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 0,
          "labels": {},
          "name": "http",
          "protocol": "tcp",
          "servicePort": 10007
        }
      ]
    },
    "labels": {
		"HAPROXY_DEPLOYMENT_GROUP":"DOTNET",
	    "HAPROXY_DEPLOYMENT_ALT_PORT":"10008",
	    "HAPROXY_GROUP":"external",
	    "HAPROXY_0_REDIRECT_TO_HTTPS":"false",
		"HAPROXY_0_VHOST": "34.215.17.237"
    },
    "networks": [
      {
        "mode": "container/bridge"
      }
    ],
    "portDefinitions": []
  }

```
###### The most important parameters are: 
- `"HAPROXY_GROUP":"external"`	- Only apps with the label _HAPROXY_GROUP=external_ will be exposed using this Marathon-LB configuration
- `"HAPROXY_0_VHOST": "34.215.17.237"`	- Add an entry for _HAPROXY_0_VHOST_ and assign  the value of your public agent IP.

###### To get public IP public node
- _With DC-OS web interface:_

Go to tab `Node` -> find colum `Type` & `Public IP`-> find line `Public` -> use `Public IP` in `Public` line.

- _With command line:_

```bash
dcos node list --json | jq --raw-output '.[] | select(.attributes.public_ip == "true") | .public_ips'

[
  "34.215.17.237"
]
```
### 4. Add service:
- _With DC-OS web interface:_

Go to tab `Services` -> press button (+) `Run a service` -> chose `JSON Configuration` -> paste json configuration -> press button `Revew & Run`

- _With command line:_

```bash
dcos marathon app add ./dotnet-website.json
```

### 5. Check result
- _With DC-OS web interface:_

Go to tag `Services` -> click on service `marathon-lb` -> go to tab `logs` -> click button `Output (stdout)`

You should find next information:

```
2019-06-25 11:38:26,033 marathon_lb: configuring app /DOTNET
2019-06-25 11:38:26,033 marathon_lb: frontend at *:10007 with backend DOTNET_10007
2019-06-25 11:38:26,033 marathon_lb: adding virtual host for app with hostname 34.215.17.237
2019-06-25 11:38:26,033 marathon_lb: adding virtual host for app with id /DOTNET
2019-06-25 11:38:26,034 marathon_lb: backend server 172.16.2.241:31030 on 172.16.2.241
2019-06-25 11:38:26,034 marathon_lb: backend server 172.16.2.241:31090 on 172.16.2.241
2019-06-25 11:38:26,034 marathon_lb: backend server 34.221.25.108:31201 on 34.221.25.108
2019-06-25 11:38:26,034 marathon_lb: backend server 34.221.25.108:31756 on 34.221.25.108
2019-06-25 11:38:26,034 marathon_lb: backend server 34.221.25.108:31779 on 34.221.25.108
```
Folow the link in your browser: http://34.215.17.237:10007