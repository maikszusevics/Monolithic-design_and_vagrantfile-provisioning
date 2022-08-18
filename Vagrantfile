#vagrant

Vagrant.configure("2") do |config|


    config.vm.define "db" do |db|
        db.vm.box = "ubuntu/xenial64"
        db.vm.network "private_network", ip: "192.168.10.150"
        db.vm.synced_folder ".", "/home/vagrant/monger"
        db.vm.provision "shell", inline: <<-SCRIPT
            # updates ubuntu
            sudo apt-get update
            sudo apt-get upgrade -y
        
            # retrieves key from mongodb
            sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927
            echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
        
            # updates ubuntu
            sudo apt-get update
            sudo apt-get upgrade -y
        
            sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20
        
            # enables and starts mongodb
            sudo systemctl restart mongod
            sudo systemctl enable mongod
            cd monger
            sudo cp -f monconfig /etc/mongod.conf
            sudo systemctl restart mongod
            
            

            SCRIPT
    end

    config.vm.define "app" do |app|
        app.vm.box = "ubuntu/xenial64" # Linux - ubuntu 16.04
        # creating a virtual machine ubuntu 
        app.vm.network "private_network", ip:"192.168.10.100"
        #once you added private network, need to reboot vm
        app.vm.synced_folder ".", "/home/vagrant/app"
        # sync data from localhost 
        # Shell provisioning to handle all dependencies and automate app start
        app.vm.provision "shell", inline: <<-SCRIPT
            sudo apt-get update && sudo apt-get upgrade -y
            sudo apt-get install nginx -y
            sudo systemctl enable ngninx 
            curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash - 
            sudo apt-get install -y nodejs
            sudo cp -f app/app/reverse_proxy /etc/nginx/sites-available/default
            sudo systemctl restart nginx
            echo "DB_HOST=mongodb://192.168.10.150:27017/posts" | sudo tee -a /etc/environment
            cd app/app
            
            cd seeds
            sudo node seed.js

            sudo apt-get update
            sudo apt-get upgrade -y
           
            npm install pm2 -g -y
            npm install express -y
            npm install mongoose -y
           

            SCRIPT
    end


end
