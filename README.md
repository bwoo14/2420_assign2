# 2420_assign2

By Brandon Woo

http://24.199.69.253/ <br>
http://24.199.69.253/api

- [2420_assign2](#2420-assign2)
  * [Tutorial](#tutorial)
    + [Step 1: DO Setup](#step-1--do-setup)
    + [Step 2: Create 2 users](#step-2--create-2-users)
    + [Step 3: Installing Web Servers](#step-3--installing-web-servers)
    + [Step 4: Writing the webapp](#step-4--writing-the-webapp)
    + [Step 5: Make the Caddyfile](#step-5--make-the-caddyfile)
    + [Step 6: install volta on droplets](#step-6--install-volta-on-droplets)
    + [Step 7: Write the node service file](#step-7--write-the-node-service-file)
    + [Step 8: Moving the node service file](#step-8--moving-the-node-service-file)
    + [Step 9: Testing](#step-9--testing)

## Tutorial

### Step 1: DO Setup
- Create 2 Droplets, a VPC, a Load Balancer, and a Firewall.
- The firewall should be configured to allow HTTP traffic on port 5050 to the load balancer
![](./images/droplets.png)
![](./images/vpc-img.png)
![](./images/load-bal-img.png)
![](./images/firewall-img.png)

### Step 2: Create 2 users
- SSH into both droplets and create a user for each of them
- you can use the same name and same password for both
![](./images/first_user.png)
![](./images/second_user.png)

### Step 3: Installing Web Servers
- install Caddy on both droplets
- make sure to run these 2 commands before installing
```
sudo apt update
sudo apt upgrade
```
- update both droplets
![](./images/apt-update-d1.png)
![](./images/apt-update-d2.png)
- upgrade both droplets
![](./images/apt-upgrade-d1.png)
![](./images/apt-upgrade-d2.png)

- to begin the installation caddy, run the command
```
wget https://github.com/caddyserver/caddy/releases/download/v2.6.2/caddy_2.6.2_linux_amd64.tar.gz
```
- this will put a tar file in your home directory
![](./images/wget-caddy.png)
- Unarchive the tar.gz file with
```
tar xvf caddy_2.6.2_linux_amd64.tar.gz
```
![](./images/tar.png)
- change the owner of the caddy gile and group to root
```
sudo chown root: caddy
```
![](./images/chown.png)
- copy the caddy file to the bin directory
```
sudo cp caddy /usr/bin/
```
![](./images/copy.png)

<br>**MAKE SURE TO DO THIS ENTIRE STEP ON BOTH DROPLETS**

### Step 4: Writing the webapp
- On your local machine, create a new directory
```
mkdir 2420-assign-two
```
![](./images/mkass2.png)
- Create two directories called `html` and `src`
```
mkdir html && mkdir src
```
![](./images/htmlandsrc.png)
- Inside html directory create an `index.html` page and add some content to it
```
cd html
vim index.html
```
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Cookies</title>
</head>
<body>
    <h1>Cookie Stealer</h1>
    <ul>
        <li>Cookie 1 is mine</li>
        <li>Cookie 2 is mine</li>
        <li>Cookie 3 you can keep</li>
    </ul>
</body>
</html>
```
![](./images/vimindex.png)
- Inside of the src directory run the following command
```
npm init
```
- follow the instructions to create the npm files
![](./images/npm-init.png)
- now run
```
npm i fastify
```
![](./images/npm-fastify.png)

- create an `index.js` file and put the following content in it
```
// Require the framework and instantiate it
const fastify = require('fastify')({ logger: true })

// Declare a route
fastify.get('/', async (request, reply) => {
  return { hello: 'Server x' }
})

// Run the server!
const start = async () => {
  try {
    await fastify.listen({ port: 3000 })
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()
```
![](./images/indexjs.png)
![](./images/vimindexjs.png)
- Install Volta then install node using
```
curl https://get.volta.sh | bash
source ~/.bashrc
volta install node
which npm
```
- It should look like this
![](./images/installvolta.png)
- Test the server with
```
node index.js
```
- and follow the link, it should look like this
![](./images/successtest.png)
- after testing change the index.js file to contain the right route and the right port (`/api`, and `5050`)
```
// Require the framework and instantiate it
const fastify = require('fastify')({ logger: true })

// Declare a route
fastify.get('/api', async (request, reply) => {
  return { hello: 'Server x' }
})

// Run the server!
const start = async () => {
  try {
    await fastify.listen({ port: 5050 })
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()
```
- move the files to both of your droplets with sftp
```
sftp -i "~/.ssh/DO_key" ocean@165.232.159.95
put 2420-assign-two
```
or

```
rsync -r 2420-assign-two "ocean@165.232.159.95:~/" -e "ssh -i ~/.ssh/DO_key -o StrictHostKeyChecking=no"
```
![](./images/sftp.png)

### Step 5: Make the Caddyfile
- make a caddyfile with
```
vim Caddyfile
```

-  add the following content
```
http:// {
    root * /var/www/html
    reverse_proxy /api localhost:5050
    file_server
}
```
![](./images/vimcad.png)
![](./images/caddyfilecontent.png)
- upload the caddy file to **BOTH** droplets using sftp
![](./images/caddysftp.png)
- On each droplet: move the caddy script we installed earlier to /usr/bin
![](./images/copyscript.png)
- make a directory in /etc called caddy
```
sudo mkdir /etc/caddy
```
![](./images/mkdircaddy.png)
- move the Caddyfile we made to the /etc/caddy directory
```
sudo mv Caddyfile /etc/caddy
```
![](./images/mvcaddy.png)

- create a service file for the caddy server
```
vim caddy.service
```
- and put this inside
```
[Unit]
Description=Serve HTML in /var/www using caddy
After=network.target

[Service]
Type=notify
ExecStart=/usr/bin/caddy run --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```
![](./images/cadserv.png)
- move the file to the /etc/systemd/system
```
sudo mv caddy.service /etc/systemd/system
```
![](./images/mvserv.png)

- restart, start, and enable the service
```
sudo systemctl daemon-reload
sudo systemctl start caddy.service
sudo systemctl enable caddy.service
```
![](./images/reload.png)

- make a directory called `www` in the `/var` directory with
```
cd /var && mkdir www
```
- move the `src` and the `html` directory we uploaded to the `/var/www` directory
```
sudo mv ./2420-assign-two/src /var/www
sudo mv ./2420-assign-two/html /var/www
```
![](./images/mvsrc.png)
![](./images/mvhtml.png)

### Step 6: install volta on droplets

- Install Volta then install node using
```
curl https://get.volta.sh | bash
source ~/.bashrc
volta install node
volta install npm
which npm
```
- It should look like this
![](./images/insatllvoltaserver.png)

### Step 7: Write the node service file
- Create a service file called `hello_web.service` on your local machine and add the following content:
```
[Unit]
Description=node application service file
After=network.target

[Service]
Type=simple
User=ocean
Group=ocean
ExecStart=/home/ocean/.volta/bin/node /var/www/src/index.js
Restart=on-failure
SyslogIdentifier=hello_web

[Install]
WantedBy=multi-user.target
```
### Step 8: Moving the node service file
![](./images/nodeserv.png)
- put this file on both droplets using sftp and move it to the `/etc/systemd/system`
![](./images/nodeservmv.png)
- reload the daemon again with
```
sudo systemctl daemon-reload
sudo systemctl start hello_web.service
sudo systemctl enable hello_web.service
```
- check that the service is working with
```
sudo systemctl status hello_web.service
```
![](./images/statushello.png)
- do this on both droplets
- Make sure to change one of the html documents so that you can tell if your load balancer is working
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Cookies</title>
</head>
<body>
    <h1>Cookie Stealer</h1>
    <ul>
        <li>Cookie 1 is mine</li>
        <li>Cookie 2 is mine</li>
        <li>Cookie 3 you can keep</li>
    </ul>
    <p>THIS IS SERVER 2</p>
</body>
</html>
```
- Also change the index.js file so that it is slightly different in the `/api` route. For example you can change the `return {hello: 'Server x'}` to `return {hello: 'Server x'}`
![](./images/servery.png)
- Make sure to do restart the web service after making changes to the html or js
```
sudo systemctl restart hello_web.service
```
![](./images/web_serv_res.png)
### Step 9: Testing
- Test that you can connect to the server, it should look like this
<br>![](./images/firstsuccess.png)
- if you hit refresh multiple times, it should at some point look different
<br>![](./images/secondsuccess.png)
- the api route should look like this
<br>![](./images/apisuccess.png)
<br>![](./images/apiservery.png)
