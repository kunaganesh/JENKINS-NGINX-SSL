
# JENKINS + NGINX + SSL
## Jenkins Installation :
* First update the repo :
```bash
 sudo apt update 
```
* Next install the jdk :
```bash
 sudo apt-get install openjdk-11-jdk -y
```
 * Add the keys :
 ```bash
   curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```
* Then add a Jenkins apt repository entry :
```bash
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
  
```
* Now install the jenkins :
```bash
 sudo apt upadte 

 sudo pat install jenkins -y
```
* Start and enable the jenkins service :
```bash
 systemctl start jenkins

 systemctl enable jenkins 
```
* Now our jenkins running plz chceck it :
```bash
 http://publicip:8080
```
* Now Map the ip address to a domain name from Route53 like dev.jenkins.com
* Now our jenkins running on domain name :
```bash
 http://dev.jenkin.com
```
## Nginx Reverse Proxy For Jenkins :
* First update the repo :
```bash
 sudo apt-get upadte 
```
* Install the nginx server :
```bash
 sudo apt install nginx-full -y
```
* To see the version :
```bash
 nginx -v
```
* Next go to the directory :
```bash
 cd /etc/nginx/sites-available
```
* Now create a file with your domain name :
```bash
 touch dev.jenkins.com
```
* Open that file with vim editor :
```bash
 sudo vim dev.jenkins.com
```
* Now put the below content in that file :
```bash
 
server {
    server_name dev.jenkins.com;

    location / {
      proxy_set_header      Host $host:$server_port;
      proxy_set_header      X-Real-IP $remote_addr;
      proxy_set_header      X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header      X-Forwarded-Proto $scheme;

      proxy_pass        http://127.0.0.1:8080;
    }
}
```

# OR Use the following conf file :
```bash
 server {
    listen 80;
    server_name dev.jenkins.com;

	location / {
		include /etc/nginx/proxy_params;
		proxy_pass          http://localhost:8080;
		proxy_read_timeout  60s;
        # Fix the "It appears that your reverse proxy set up is broken" error.
        # Make sure the domain name is correct
		proxy_redirect      http://localhost:8080 https://dev.jenkins.com;
	}
}
```
* Next create a symbolic link to sites enabled :
```bash
 sudo ln -s /etc/nginx/sites-available/dev.jenkins.com /etc/nginx/sites-enabled/dev.jenkins.com
```
* Now restart the nginx :
```bash
 sudo systemctl restart nginx 
```
* Now our jenkins will be runing via nginx lets check :
```bash
 http://dev.jenkins.com
```
* By default Jenkins listens on all network interfaces. But we need to disable it because we are using Nginx as a reverse proxy and there is no reason for Jenkins to be exposed to other network interfaces.
* We can change this by editing /etc/default/jenkins :
```bash
 cd /etc/default
```
* Open the jenkins file :
```bash
 sudo vim jenkins 
```
* Add two lines as shown below :
![Screenshot (1)](https://user-images.githubusercontent.com/98937778/211758824-b6a2fdd3-a26a-4e05-9168-e9337ff8e030.png)

* now restart the jenkins :
```bash
 systemctl restart jenkins
```
* Now our jenkins running only on standard port 80 

## HTTPS Configuration :
* First let run the following command :
```bash
 sudo apt-get install python3-certbot-nginx
```
* Run the following command to recieve our certificates :
```bash
 sudo certbot certonly --nginx 
```
* After execute this command it will ask for email enter it :
* Next enter A to agree 
* Next enetr Y to yes
* Then it automatically detects your domain which is mentioned in nginx file 
* Now just enter 

* Next run the command to install certificates :
```bash
 sudo certbot install --nginx 
```
* Now take a look on sites-enabled directory dev.jenkins.com file ..certbot did some changes for us 
* Now restart the nginx server :
```bash
 sudo systemctl restart nginx 
```
* Now our jenkins running on asecure connection with nginx .


# Process - 2
* Setup the Jenkins.
## Step 1 - Configuring Nginx
```bash
sudo apt-get update

sudo apt-get install nginx

```
## Step 2 - Getting a Certificate
* After that, you will need to purchase or create an SSL certificate. The commands we are showing are for self-signed certificates, to avoid any browser warnings you should get a signed certificate.

* Next, move into the proper directory and generate a certificate:
```bash
 cd /etc/nginx

 sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/cert.key -out /etc/nginx/cert.crt

```
* You will now be asked to enter some information about the certificate. You can fill this out however you wish, just keep in mind that the information will be visible in the certificate properties. You will be setting the number of bits to 2048 since that is the minimum needed to get it signed by a CA. If you wish to get the certificate signed, you will need to create a CSR.
## Step 3 - Editing the Configuration
* After that, you will need to edit the default Nginx configuration file:
```bash
 sudo nano /etc/nginx/sites-enabled/default

```
* You will get a final configuration as follows. You can also update or replace the existing config file, although you may want to first make a quick copy:
```bash
 server {
    listen 80;
    return 301 https://$host$request_uri;
}

server {

    listen 443;
    server_name jenkins.domain.com;

    ssl_certificate           /etc/nginx/cert.crt;
    ssl_certificate_key       /etc/nginx/cert.key;

    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    access_log            /var/log/nginx/jenkins.access.log;

    location / {

      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      # Fix the “It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://localhost:8080;
      proxy_read_timeout  90;

      proxy_redirect      http://localhost:8080 https://jenkins.domain.com;
    }
  }
```
* In your configuration, the cert.crtand cert.key settings reflect the location where you have created your SSL certificate. You will need to update the servername and proxyredirect lines with your own domain name.
## Step 4 - Configuring Jenkins
* For Jenkins to work with Nginx, you need to update the Jenkins config to listen only on the localhost interface instead of all (0.0.0.0), to ensure traffic gets handled properly. This is a crucial step because if Jenkins is still listening on all interfaces, then it will still potentially be accessible via its original port (8080). We will edit the /etc/default/jenkins configuration file to make these adjustments.
```bash
 sudo nano /etc/default/jenkins

```
* After that, locate the JENKINS\_ARGS line and update it to look something like this:
```bash
 JENKINS_ARGS="--webroot=/var/cache/jenkins/war --httpListenAddress=127.0.0.1 --httpPort=$HTTP_PORT -ajp13Port=$AJP_PORT"

```
* After that, go ahead and restart Jenkins and Nginx :
```bash
 sudo service jenkins restart

 sudo service nginx restart

```
* And you will get an error in manage jenkins dashboard like this :
![image](https://user-images.githubusercontent.com/98937778/211993018-4a177306-7675-45a4-8603-256f4dde2283.png)

* Jenkins -> Manage Jenkins -> Configure System -> Jenkins Location

* After that, update the Jenkins URL to use HTTPS - https://jenkins.domain.com/

* As a next step, update your OAuth settings with the external provider. This example is for GitHub. On GitHub, you can find it under Settings -> Applications -> Developer applications, on the GitHub site.
![image](https://user-images.githubusercontent.com/98937778/211993425-1e773f09-78fa-47dc-9447-2be9e2bd0b21.png)

## Thats it !
