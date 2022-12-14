# chef-tutorial
* in this tutorial i used an amazon instance ubtuntu image and type t2.medium
* when you create your ec2 instances where you will install the chef workstation, server, node make sure to enable below security group
![sg](https://user-images.githubusercontent.com/67081878/185626816-f8d23b72-4b40-4807-9603-8b81686eb41a.PNG)

## chef server
* install chef server
```
wget https://packages.chef.io/files/stable/chef-server/13.1.13/ubuntu/18.04/chef-server-core_13.1.13-1_amd64.deb
sudo dpkg -i chef-server-core_*.deb
sudo chef-server-ctl reconfigure
```
* create .chef directory on the home directory
```
mkdir $HOME/.chef
```
* create user and organization
```
sudo chef-server-ctl user-create abdelali abdelali jadelmoula jadelmoulaa2@gmail.com 'Aa123456789@' --filename /home/ubuntu/.chef/abdelali.pem

sudo chef-server-ctl org-create org abdelali_org --association_user abdelali --filename /home/ubuntu/.chef/abdelali_org.pem
```

* install the console management to interact with the chef server through the browser
```
chef-server-ctl install chef-manage
chef-server-ctl reconfigure
chef-manage-ctl reconfigure
```
* install the workstation
```
wget  https://packages.chef.io/files/stable/chef-workstation/0.2.43/ubuntu/18.04/chef-workstation_0.2.43-1_amd64.deb
sudo dpkg -i chef-workstation_*.deb
```
* generate ssh keys and copy the public one on the chef server
```
ssh-keygen -t rsa

ssh-copy-id username-of-chef-ser@chef-ip-address
```

* create directories on the workstation
```
mkdir -p chef-repo/.chef
```

* copy the chef user and chef organization pem files from .chef on the chef server to the workstation
scp chef-server-user@chef-server-ip-address:~/.chef/*.pem ~/chef-repo/.chef

* creare config.rb file under /chef-repo/.chef (this file will have creadentials and instruction that will be used by knife)
```
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                'node_name'
client_key               "USER.pem"
validation_client_name   'ORG_NAME-validator'
validation_key           "ORGANIZATION-validator.pem"
chef_server_url          'https://example.com/organizations/ORG_NAME'
cache_type               'BasicFile'
cache_options( :path => "#{ENV['HOME']}/.chef/checksums" )
cookbook_path            ["#{current_dir}/../cookbooks"]
```
* example:
```
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                'abdelali'
client_key               "abdelali.pem"
validation_client_name   'abdelali_org-validator'
validation_key           "abdelali_org-validator.pem"
chef_server_url          'https://ip-16-10-0-102.eu-west-3.compute.internal/organizations/abdelali_org'
cache_type               'BasicFile'
cache_options( :path => "#{ENV['HOME']}/.chef/checksums" )
#cookbook_path            ["#{current_dir}/../cookbooks"]
cookbook_path            ["#{current_dir}/"]
```
* the node_name is the username you have created on the chef server with the organization

* make sure to run the knife command on the directory where your .chef directory exists

* run the below command

knife ssl fetch

* check wether if everything work as expected
```
knife ssl check  
```

* if you see errors after your runned the above command, make sure that the name of the chef server (ip-16-10-0-102.eu-west-3.compute.internal)
* is the same as the name of the certificate that is been generated when you runned the knife ssl fetch command
* to check that go to $HOME/chef-repo/.chef/trusted_certs and see the name of *.crt certificat because this certificate is was signed
* for this ip-16-10-0-102.eu-west-3.compute.internal and if you try to use another host name for your chef server then you will face errors


## create chef-node-server 

* join the node to the chef server
```
 knife bootstrap 192.0.2.0 -x root -P password --node-name nodename
 ```
* make sure to use the root credentials because this command will install some packages 
* on your chef node server so those commands will require priviledges

## testing

* create a director called cookbooks under chef-repo/.chef

```
mkdir $HOME/chef-repo/.chef
```

* create chef cookbook 
```
chef generate cookbook sample

```
* edit sample/recipies/default.rb

```
nano sample/recipies/default.rb
```

* add the below lines to the default.rb file
```
package 'nginx'
service 'nginx' do
action [:enable, :start]
end
```

* this will install and start the nginx web server service 

* upload the cookbook to chef server 
```
knife cookbook upload sample
```

* add run list to your node  either from the console or using the command 

* click on the icon on the right side of the setting icon and click on the Edit Run List option
![chef1](https://user-images.githubusercontent.com/67081878/185625405-78423a3a-36df-48b0-aab2-48177e25e628.PNG)

* this pop-up will appear 
![chef2](https://user-images.githubusercontent.com/67081878/185625408-3049fe27-d536-4593-8715-dadce1688e83.PNG)
* drag and drop the recipe from the Available recipe section
![chef3](https://user-images.githubusercontent.com/67081878/185625411-a195795a-0cd4-4d8c-9b7b-21309c4f8a7b.PNG)
* click on the save button
![chef4](https://user-images.githubusercontent.com/67081878/185625417-630e8276-1e91-4410-b999-816f6b36682c.PNG)

* then go to your chef node server and run chef-client command 
```
sudo chef-client 
```

* this command will pull the cookbook and run the recipies



