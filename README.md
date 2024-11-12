# wordpress-server-docker
WordPress deployment with Docker Compose and Ngrok tunneling

This is a continuation of the DevOps series, now deploying the server using Docker Compose. 

- [Part 1](https://github.com/Burush0/wordpress-ubuntu-server) (raw step by step)
- [Part 2](https://github.com/Burush0/wordpress-server-bash) (bash script)
- [Part 3](https://github.com/Burush0/wordpress-server-ansible) (Ansible playbook with separate roles).

# Prerequisites:
- An [Ubuntu Server](https://ubuntu.com/download/server) installation, preferably fresh. I did mine inside a VMWare Workstation virtual machine.
- A way to generate SSH keys on your host machine. In my case of Windows 10, I had [Git](https://gitforwindows.org/) installed, that can also be achieved with [PuTTY](https://www.putty.org/).
- An [Ngrok](https://ngrok.com/) account.

# Setup:
## 1. Setup SSH connection between guest (VM) and host (your PC):

### 1.1. Install OpenSSH Server:

```
sudo apt install openssh-server -y
```

### 1.2. Generate an SSH key on your host machine:
```
ssh-keygen -t rsa -b 4096
```

### 1.3. Send the public part of the key to the guest. 

```
ssh-copy-id <remote-user>@<server-ip>
```

### 1.4. (Optional but highly recommended) VSCode with SSH

Since we will be working with multiple directories and regularly updating and editing files, I recommend using [Visual Studio Code](https://code.visualstudio.com/) with [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension installed. This way, you will be able to SSH right into the working directory of the server and work inside it as you would on the host machine, without needing to needlessly navigate and edit everything with terminal commands and terminal editors.

## 2. Specify a local domain name

In `/etc/hosts` file on your system (For Windows, that would be `C:/Windows/System32/drivers/etc/hosts`), add in a line that looks like this:
```
<server-ip> <domain-name>
```
To give a more specific example, my local server ip was `192.168.139.134` and the domain name I chose was `testdomain.com`, so the line looked like:
```
192.168.139.134 testdomain.com
```
Save changes to the file, needs administrator privileges (which means you might need to run Notepad as admin).

## 3. Install Docker, Docker Compose and Ngrok.

I opted against using Docker Desktop and went with Docker Engine and a standalone Compose installation. I am not sure if Docker Desktop will work as intended on a Ubuntu Server as opposed to regular Ubuntu.

I was going to write it out, but there are officially available guides on their respective websites, you should be able to follow them. Remember that since you SSH over to a Ubuntu machine, you should follow the guide for Linux, not your host OS. For Docker, I ran post-installation steps that made me able to run docker commands without specifying sudo each time. Just a personal preference because it's convenient.

## 4. Run the compose installation

### 4.1. Clone the repo and navigate to the folder
```
git clone https://github.com/Burush0/wordpress-server-docker.git
cd wordpress-server-docker
```
### 4.2. Create a `.env` file to specify variables for the MySQL database

In my case, it looks like this:
```
MYSQL_ROOT_PASSWORD=root
MYSQL_USER=wordpress
MYSQL_PASSWORD=qwe123
```

### 4.3. Generate the OpenSSL certificates and DH params, those will be sent over to the Nginx container to allow HTTPS connections

Run the following commands (as per usual, if you want your own information, change the `-subj` part of the first command):
```
sudo openssl req -x509 -noenc -days 365 -newkey rsa:2048 -keyout ./ssl/private/nginx-selfsigned.key -out ./ssl/certs/nginx-selfsigned.crt -subj "/C=SE/ST=Vaestra Goetalands laen/L=Goeteborg/O=Burush Inc. /OU=DevOps Department/CN=Burush"
sudo openssl dhparam -out ./ssl/dhparams.pem 2048
```

### 4.4. Edit the `ngrok/ngrok.yaml` config file

There will be a line which says `YOUR_TOKEN_HERE`, guess where the token goes? You can obtain your token on the Ngrok dashboard [here](https://dashboard.ngrok.com/get-started/your-authtoken).

As for the domain name, you can find your own in the "Setup & Installation" tab of the dashboard, after scrolling down a bit. You're looking for a static domain, mine is `tight-badger-oddly.ngrok-free.app`, yours will be different

### 4.5. Run the compose.yaml file
```
docker compose -f compose.yaml up -d 
```

And that's it! You should have a working WordPress deployment at `https://<SERVER_NAME>/wordpress/`, as well as having it accessible to the internet through the Ngrok tunnel at `https://<ngrok-static-domain>/wordpress`. If you're getting a 403 error from Nginx at `https://<ngrok-static-domain>`, that is to be expected, no endpoint is specified at `/`.
