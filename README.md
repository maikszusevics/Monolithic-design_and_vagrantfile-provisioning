# Deployment
### Monolithic architecture
A monolithic architecture of a software program refers to a unified application that is self-contained and independent from other applications.


![Blank board - Page 1 (1)](https://user-images.githubusercontent.com/110176257/184701206-059dceb1-7751-4f5e-993e-513e1a811a31.png)

## Vagrant provisioning
A provisioner is configured inside the Vagrantfile using the `config.vm.provision` method.
The shell provisioner can be used in two ways, inline and path.
```ruby
config.vm.provision "shell", inline:
```
```ruby
config.vm.provision "shell", path:
```
### Inline
The `inline` method specifies the provisioning shell commands direcly after `inline:` or in a script. 

The script block starts with `<<-SCRIPT` and ends with `SCRIPT`

In the example below, the Vagrantfile utilises the `inline` method of the shell provisioner to automate all of the dependencies needed for the SpartaGlobal app, and to start the app automatically.
```ruby
Vagrant.configure("2") do |config|

  # Shell provisioning to handle all dependencies and automate app start
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
```

### Path
Path to a shell script to execute:

In order to use a shell file, you would need to create a shell script file in the same directory as the Vagrantfile.

You can create a `.sh` file by navigating to the diractory in a terminal and then running `nano script.sh`

My locahost vagrant directory is called "vagrant":

![image](https://user-images.githubusercontent.com/110176257/184713983-7f92182a-cec1-4249-9437-34f1c1882645.png)

Running `nano script.sh` will open the nano editor:

![image](https://user-images.githubusercontent.com/110176257/184715430-e65e47cd-c927-450b-ad8c-e35bf416c521.png)

here you can enter the provisioning commands, just like in the script block when using the inline method:
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install nginx -y
sudo systemctl enable ngninx 
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash - 
sudo apt-get install -y nodejs
npm install pm2 -g -y
npm install express -y
npm install mongoose -y
```
Then you can press ctrl+x and y to save and exit.
Check the file is there by running `ls`

Now you can use the file by adding the following lines into the vagrantfile:
```ruby
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "script.sh"
```

## Running `vagrant up` command with dependency and app launch automation script in the Vagrantfile

![image](https://user-images.githubusercontent.com/110176257/184716378-61f2dfa5-4b8e-4281-b6ba-4d5ec48a59eb.png)

In the above image, you can see the inline method being used with a provision script, and that the final output of `vagrant up` command stated the application is up and running. 

You can check if the provisioning has worked by running the tests contained within the evnironment/spec-tests by using `rake spec`

![image](https://user-images.githubusercontent.com/110176257/184719007-4a050cf7-266b-4373-bac2-78eeb547e8bc.png)

All tests passed, provisioning script works.

We can check by going to the application url at http://192.168.10.100:3000/

![image](https://user-images.githubusercontent.com/110176257/184716733-f4ffd20b-d7ae-40f9-b1ac-cac483347f9c.png)

## Reverse proxy and automating reverse proxy

A reverse proxy is an intermediary server between clients and backend servers. Reverse proxies make reaching and navigating websites and web applications a lot more user friendly because of the additional level of abstraction it provides to the end user (e.g not having to know or input the correct port number). In this example we will be setting up a reverse proxy to direct client requests from port 80 to the appropriate backend server (port 3000 in this case). 

In order to set up a reverse proxy, we must edit the nginx configuration at `etc/nginx/sites-available/default`
To redirect port 80 to port 3000 use `sudo nano /etc/nginx/sites-available/default`

![image](https://user-images.githubusercontent.com/110176257/184970952-3ec3728a-ae1d-4d6e-9a68-9333049c6cdd.png)

the above image is what you will see once you enter the default file, from here you have to simply scroll to `location / {` and replace what's there with `proxy_pass http://localhost:3000;` 

Example:

![image](https://user-images.githubusercontent.com/110176257/184971650-88259e35-6f88-4a81-8cbf-136d6567456a.png)

Is changed to:

![image](https://user-images.githubusercontent.com/110176257/184971805-37d439f9-a5d7-4a13-9844-e47b7aa8716d.png)

Now you can run `sudo nginx -t` to check syntax:

![image](https://user-images.githubusercontent.com/110176257/184972177-a060dd55-b603-4977-8488-bcf7279a7136.png)

If it's okay, reload nginx with `sudo systemctl restart nginx` and the reverse proxy is set up and running.

## Automating reverse proxy in the provision script

To automate your reverse proxy settings: you will need to create a new file with the configuration, and add a command to the provisioning script which overwrites the default configuration with what you put inside your file.

First choose the location this new file will be in, this example uses the app/app folder of the VM.
- Move to the desired directory with `cd`, 
- Create a file using `sudo nano rev_proxy`
- Write your reverse proxy configuration in the empty rev_proxy file

To redirect port 80 to port 3000 all we need in our config file is this:
```nginx
server {
        listen 80;
        listen [::]:80;

        access_log /var/log/nginx/reverse-access.log;
        error_log /var/log/nginx/reverse-error.log;

        location / {
                    proxy_pass http://localhost:3000;
  }
}
```
##### Now we can automate this file overwriting the `/etc/nginx/sites-available/default` file.

Add these lines to the end of your provisioning script:

```
sudo cp -f app/app/rev_proxy /etc/nginx/sites-available/default

sudo systemctl restart nginx

```
The `sudo cp -f app/app/rev_proxy /etc/nginx/sites-available/default` command is copying the `app/app/rev_proxy` directory to the `/etc/nginx/sites-available/default` directory. The `-f` signifies to overwrite.

The next command orders to restart nginx service which will apply the changes. 

Here you can see these commands inside my provisioning script:

![image](https://user-images.githubusercontent.com/110176257/184974953-f0ed24f2-b106-4cad-808e-8762cb43a2fb.png)

# Multiple machines

In order for the Sparta Test App to work, we need a seperate database VM.
We can use the Vagrantfile to make multiple machines.

![image](https://user-images.githubusercontent.com/110176257/184979350-90efa685-ed84-4772-b737-cab3843c07b6.png)

The above image shows how we can edit the Vagrantfile to create a VM called "app".
Within the app configuration there is included the provisioning script we used above to automate dependencies and reverse proxy setup.

Below that provisioning script, you use 
```ruby
config.vm.define "db" do |db|
        db.vm.box = "ubuntu/xenial64"
```
to configure the database machine:


![image](https://user-images.githubusercontent.com/110176257/184980019-0613f782-8f35-4a89-a116-62c66bd05aa8.png)

Now when you run `vagrant up`, and then `vagrant status`, you should see there are two machines running.

![image](https://user-images.githubusercontent.com/110176257/184980295-1f9ac8a6-d24f-4a43-8c2a-ef965e8aa432.png)

If only one of the machines is running, you can run `vagrant up db` to specify.

## Persistent env variables
For app to communicate with db, it will need a `DB_HOST` environmental variable.
To create one that doesnt disappear after exiting the machine we must edit the `.bashrc file` in the home folder.

Use `sudo nano .bashrc` from the default directory to open the editor, then add `export DB_HOST=mongodb://192.168.10.150:27017/posts` on the *end of the file*

use source command to make sure changes are scanned and applied:
`source ~/.bashrc`

Now with all this set up, this is how our configuration should look like:

![image](https://user-images.githubusercontent.com/110176257/184983030-d9776609-642e-4c3c-86f9-65cbb7ca12c0.png)


# Creating mongodb database on db machine and enabling /posts functionality

![image](https://user-images.githubusercontent.com/110176257/185150435-c50e59d7-dd6e-467f-82b2-98d59d489046.png)

## Installing mongodb on db machine 

- Make sure db is running by `vagrant status`, if its not running run `vagrant up db`
- Connect to db with `vagrant ssh db`
- Run `sudo apt-get update && sudo apt-get upgrade -y`
- Run `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927`
- Run `echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list`
- Run `sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20`
- Run `sudo systemctl start mongod`
- To make sure it is running, run `sudo systemctl status mongod`
- It should say `Active: active (running)`

- Now you will have to edit the mongodb configuration file.
- You can do this by running `sudo nano /etc/mongod.conf`
- 
In this file, you can scroll down until you see the IP, and change it to the IP you wish to be able to connect, or change it to all zeroes for anyone to be able to connect. Although in a real work scenario all zeroes would be a major security issue.

Save the file by control x and y 

- run `sudo systemctl restart mongod`






