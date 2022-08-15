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

### Running `vagrant up` command with dependency and app launch automation script in the Vagrantfile

![image](https://user-images.githubusercontent.com/110176257/184716378-61f2dfa5-4b8e-4281-b6ba-4d5ec48a59eb.png)

In the above image, you can see the inline method being used with a provision script, and that the final output of `vagrant up` command stated the application is up and running. 

You can check if the provisioning has worked by running the tests contained within the evnironment/spec-tests by using `rake spec`

![image](https://user-images.githubusercontent.com/110176257/184719007-4a050cf7-266b-4373-bac2-78eeb547e8bc.png)

All tests passed, provisioning scripts works.

We can check by going to the application url at http://192.168.10.100:3000/

![image](https://user-images.githubusercontent.com/110176257/184716733-f4ffd20b-d7ae-40f9-b1ac-cac483347f9c.png)



