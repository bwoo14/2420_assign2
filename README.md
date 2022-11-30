# 2420_assign2

By Brandon Woo

## Tutorial

<br>**Step 1: DO Setup**
- Create 2 Droplets, a VPC, a Load Balancer, and a Firewall.
- The firewall should be configured to allow HTTP traffic to the load balancer
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
