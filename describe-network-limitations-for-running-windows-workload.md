This page describes hot to change the default parameters of ports for mesos-agent resources.

By default the mesos agent uses next ports range - 31000-32000. If you need to setup some port for your application you need to use this range. But you can change the default range. For this needed provide parameter to mesos agent  
```bash
--resources='ports:[2000-34000]' 
```

Then you can use the port from the range for your application. 

Here you can see the default parameters of port range - https://github.com/dcos/dcos-ansible/blob/feature/windows-support/roles/DCOS.agent_windows/vars/main.yml

If you need to change the range in ansible you need to use extra-vars over_resources_ports and setup the new range.
