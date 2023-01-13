# WhatsApp Chat Proxy For Dummies (using `AWS`)
Easy instructions to setup WhatsApp Chat Proxy using an `AWS (Amazon Web Services) EC2` instance. Assumption is that you have basic knowledge of how to open a shell terminal and navigate to different directories. 


### `Note: As of January 12 2023, WhatsApp Chat Proxy ONLY works for simple texting. Video/Audio Calls and Media (e.g. Images, Videos) upload/download are not proxied.`


## 1. Create an `AWS EC2` instance (its free!)

[Create an AWS account and sign in to AWS Console](https://aws.amazon.com/console/)

In your AWS console, search for `EC2` and click on it.
 
Click on `Launch instance`

### Name and tags
give it a name (How about `woman_life_freedom`?)

### Application and OS Images (Amazon Machine Image)
Select Amazon Linux (Make sure selected Amazon Machine Image (AMI) shows "Free tier eligible")

### Instance type
Select t2.micro option which shows "Free tier eligible"

### Key pair (login)
Click on "Create new key pair". Make sure RSA and .pem are selected. For name, type "new" and click on "Create key pair". This will automatically download a file (new.pem) into your machine and goes back to previous screen where you're setting up your EC2 instance. Under key pair, click on "Select" and choose your "new" key. 

### Network settings
Make sure `SSH`, `HTTPS` and `HTTP` are selected. (Advanced: later you can close `HTTP` and `HTTPS` ports and just open a `5222` port in your instance, but in order not to make things complicated I have not included that instructions here)

No other changes is needed. Now, go ahead and click on `launch instance` to create the new instance

When instance was created, click on `View all instances` and select the one that you just created. Scroll down, under `Details` tab, copy the `Public IPv4 address` number (e.g. 44.202.63.92) and save it somewhere. You'll need this address for next steps and also for entering it in your WhatsApp settings to enable proxy. 

## 2. Install required packages into your EC2 instance

You'll need to `SSH` into your instance in order to install few required packages. 
Open a shell terminal and change your directory where your `new.pem` file exists. 

If you are using an `SSH` client on a `macOS` or `Linux` computer to connect to your EC2 instance, use the following command to set the permissions of your private key file so that only you can read it.

```bash
chmod 400 new.pem
```

Now, `SSH` to your instance (please replace the IP address with yours copied from previous step. 44.202.63.92 below is just an example!) by running the following command.

```bash
ssh -i new.pem ec2-user@44.202.63.92
```
It'll ask you if you're sure, type `yes` and hit enter. Now, you've successfully SSH'd into your `AWS EC2` instance. Terminal will show you something like `ec2-user@ip-44.202.63.92`


Update the installed packages and package cache on your instance.
```bash
sudo yum update -y
```
You'll probably see something like "No packages marked for update" since you just created an instance. 

Install the most recent `Docker` Engine package.
```bash
sudo amazon-linux-extras install docker
```
It'll ask you if everything looks good, type `y` and hit enter. Wait until it installs `docker`.

Start the `Docker` service.
```bash
sudo service docker start
```
You'll probably see `Redirecting to /bin/systemctl start docker.service`

To ensure that the `Docker` daemon starts after each reboot of your `EC2` instance, run the following command:
```bash
sudo systemctl enable docker
```

Using the command below, add the `ec2-user` to the `docker` group so you can execute Docker commands without using `sudo`.
```bash
sudo usermod -a -G docker ec2-user
```

Now you'll need to reboot your instance so that changes apply. Go back to the Web browser (e.g. `Chrome`) and your AWS Console where you created your `EC2` instance:

go to instances

select your instance (by clicking on the checkmark left to it)

click on `Instance state` dropdown in the top right of the console and click on `Reboot instance` and then `Reboot`. The reboot process will take less than a minute. Just wait ~30 seconds and then go back to shell terminal.

Since you rebooted your instance, you got kicked out of `SSH` connection so you'll need to `SSH` again (Note: again, please replace the IP address of your instance with the example IP address below):
```bash
ssh -i new.pem ec2-user@44.202.63.92
```

Run the command below to make sure `docker` is installed successfully and that you're able to run it without `sudo` command:
```bash
docker info
```
It'll show you some info about `docker` (no shit?! :-D) 

Install `git`:
```bash
sudo yum install git -y
```

Install `docker-compose`:

first run 
```bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose
```

then run
```bash
sudo chmod +x /usr/bin/docker-compose
```

Run the following to make sure `docker-compose` is installed successfully.
```bash
docker-compose version
```
You'll see something like `Docker Compose version v2.15.1`


Now that you've successfully installed `docker`, `docker-compose` and `git`, go to [WhatsApp Proxy](https://github.com/WhatsApp/proxy) and follow the instructions. Please note that, you can now skip the instructions for `docker` and `docker-compose`. 

Basically, all you'll need from the `WhatsApp` instructions are the following two steps:

Cloning the repository

Building the proxy host container 

Then run the following so that the `WhatsApp Proxy` starts and keeps running even if you reboot your EC2 instance.
```bash
docker-compose -f proxy/ops/docker-compose.yml up -d
```


