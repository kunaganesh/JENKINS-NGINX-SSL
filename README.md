
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
