#vagrant

Vagrant.configure("2") do |config|



 config.vm.box = "ubuntu/xenial64" # Linux - ubuntu 16.04

# creating a virtual machine ubuntu 
config.vm.network "private_network", ip:"192.168.10.100"
#once you added private network, need to reboot vm

# lets sync our app folder from localhost to vm
config.vm.synced_folder ".", "/home/vagrant/app"
# sync data from localhost 
config.vm.provision "shell", inline: <<-SCRIPT
    sudo apt-get update && sudo apt-get upgrade -y
    sudo apt-get install nginx -y
    sudo systemctl enable ngninx 
    curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash - 
    sudo apt-get install -y nodejs
    cd app
    cd app
    npm install pm2 -g -y
    npm install express -y
    npm install mongoose -y
    npm install -y
    npm start -y
    SCRIPT
    "echo Provisioning completed, linux machine booted, application started."
    


end
