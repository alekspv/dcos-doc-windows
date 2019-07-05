***This instruction describes on how to install DC/OS on AWS cloud using MAWS tool and Terraform ***

Pre-requisites: 
	1) For Windows specifically install Git Bash
	2) For CentOS specifically install GUI part
	3) Install Terraform using https://docs.mesosphere.com/1.12/installing/evaluation/aws/
	4) Install AWS and MAWS using https://drive.google.com/drive/folders/1Voa9t0mqIx4JtAXPZygNRHfeXcMvAd4t
	
Here is examples (steps done on Virtual box machine - CentOS 7.5):

1) Skipped …

2) GUI install on CentOS (require for MAWS):
    sudo yum group list
    sudo yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
    sudo yum groupinstall "Server with GUI"
    ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target
    reboot

	3) Install Terraform: 
    wget https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip
    unzip terraform_0.11.13_linux_amd64.zip
    sudo mv terraform /usr/bin/terraform && sudo chown root:root /usr/bin/terraform && sudo chmod +x /usr/bin/terraform

Try:
    terraform --version

4.1) Install AWS : 
    sudo yum install awscli
    mkdir -p ~/dcos-tf-aws-mesowind && ~/dcos-tf-aws-mesowind
4.2) Install MAWS: 
    Download maws-linux from https://drive.google.com/open?id=1Gl8OzmwrJZcp8nzxNvX5lW7OuwuEB0ul
    chmod +x maws-linux && sudo mv maws-linux /usr/local/bin/maws
    
   eval $(maws li 966122790603_Mesosphere-PowerUser) 
   maws li -r 966122790603_Mesosphere-PowerUser &
   export AWS_PROFILE=966122790603_Mesosphere-PowerUser
   maws configure set account 966122790603_Mesosphere-PowerUser

***NOTE***: If the previous maws session expired due to some reason ( laptop reboot, maws system died) , 
please clean session file and re-establish session:
   mv .maws.session .maws.session.old
   eval $(maws li 966122790603_Mesosphere-PowerUser) 
   maws li -r 966122790603_Mesosphere-PowerUser &

Steps to Deploy DC/OS on AWS:
   start ssh-agent and add private key 
   eval "$(ssh-agent -s)" 
   ssh-add ~/.ssh/aws-meso-wind
   ssh-add -L

   upload main.tf to a working dir ( for instance, dcos-tf-aws-mesowind): 
   cd ~/dcos-tf-aws-mesowind
   eval $(maws s e 966122790603_Mesosphere-PowerUser) terraform init
   eval $(maws s e 966122790603_Mesosphere-PowerUser) terraform plan -out=plan-16may.1.out
   eval $(maws s e 966122790603_Mesosphere-PowerUser) terraform apply plan-16may.1.out
   
    Apply complete! Resources: 74 added, 0 changed, 0 destroyed.
    Outputs:
cluster-address = msos-wind-epam-16may-293370761.us-east-1.elb.amazonaws.com
masters-ips = [
    52.201.243.252
]
public-agents-loadbalancer = ext-msos-wind-epam-16may-8482ea422a5f707d.elb.us-east-1.amazonaws.com

Use following if you need destroy the cluster:
#eval $(maws s e 966122790603_Mesosphere-PowerUser) terraform destroy


Finally How to Connect? 
ssh -i /path_to_your_local/aws-meso-wind.pem centos@<ip-address>
