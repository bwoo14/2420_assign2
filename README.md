# 2420_assign2

By Brandon Woo

## Tutorial

<br>**Step 1: DO Setup**
- Create 2 Droplets, a VPC, a Load Balancer, and a Firewall.
- The firewall should be configured to allow HTTP traffic on port 5050 to the load balancer
![](./images/droplets.png)
![](./images/vpc-img.png)
![](./images/load-bal-img.png)
![](./images/firewall-img.png)

<br>**Step 2: Create 2 users**
- SSH into both droplets and create a user for each of them
- you can use the same name and same password for both
![](./images/first_user.png)
![](./images/second_user.png)

<br>**Step 3: Installing Web Servers**
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

<br>**Step 4: Writing the webapp**
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
![](./images/htmlforindex.png)
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
- we will have to change some things to this file later
- Install Volta then install node using
```
curl https://get.volta.sh | bash
source ~./.bashrc
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
