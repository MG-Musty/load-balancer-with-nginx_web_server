# LOAD_BALANCER USING HAPROXY ON NGINX WEB_SERVER


# SOLUTION GETTING THE CONFIGURATION ON YOUR NGINX WEB_SERVER

> This tutorial will teacher you how to install configure your Load_balancer using HAproxy on Ubuntu 20.04 running on a Nginx web_server.

* Make sure you `cd 92429-lb-01 server` example `ssh ubuntu@ip_address`.

### Step 1) Install HAProxy Load Balancer

```
$ sudo apt update
$ sudo apt show haproxy
```

![image](https://user-images.githubusercontent.com/106968663/213882857-4193c4a0-37eb-4f0b-90c2-19e05eabc100.png)

* But the latest long term support release is HAProxy is 2.6, So to install HAProxy 2.6, first enable PPA repository, run following command

```
$ sudo add-apt-repository ppa:vbernat/haproxy-2.6 -y
```

* Now install haproxy 2.6 by executing the following commands

```
$ sudo apt update
$ sudo apt install -y haproxy=2.6.\*
```

* Once installed, confirm the version of HAProxy installed as shown

```
$ haproxy -v
```

![image](https://user-images.githubusercontent.com/106968663/213885697-2f06bfbd-5e67-4402-95ca-b407613799de.png)


* Upon installation, the HAProxy service starts by default and listens to TCP  port 80. To verify HAProxy is running, run the command

```
$ sudo systemctl status haproxy
```

![image](https://user-images.githubusercontent.com/106968663/213885761-6a6feec9-c360-4507-9592-09973f578785.png)


* It’s recommended to enable the service to auto-start on very system reboot as shown.

```
$ sudo systemctl enable haproxy
```

### Step 2) Configure HAProxy

* The next step is to configure HAProxy to distribute traffic evenly between two web servers. The configuration file for haproxy is `/etc/haproxy/haproxy.cfg`.

* Before making any changes to the file, first, make a backup copy.

```
$ sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bk
```

* Then open the file using your preferred text editor. Here, we are using Nano

```
$ sudo nano /etc/haproxy/haproxy.cfg
```

### Haproxy configuration file is made up of the following sections:

* **global**: This is the first section that you see at the very top. It contains system-wide settings that handle performance tuning and security.
* **defaults**: As the name suggests, this section contains settings that should work well without additional customization. These settings include timeout and error reporting configurations.
* **frontend and backend**: These are the settings that define the frontend and backend settings. For the frontend, we will define the HAProxy server as the front end which will distribute requests to the backend servers which are the webservers. We will also set HAProxy to use round robbin load balancing criteria for distributing traffic.
* **listen**: This is an optional setting that enables you to enable monitoring of HAProxy statistics.

### Now define the frontend and backend settings:

```
frontend linuxtechi
   bind *:80
   model http
   stats uri /haproxy?stats
   default_backend web-servers

backend web-servers
    balance roundrobin
    server 12345-web1 10.128.0.27:80 check
    server 12345-web2 10.128.0.26:80 check
```

* In order to enable viewing the HAProxy statistics from a browser, add the following `‘listen’ section`.

```
listen stats
   bind *:8080
   stats enable
   stats uri /
   stats refresh 5s
   stats realm Haproxy\ Statistics
   #stats auth linuxtechi:P@ss123-(Not neccessary)     #Login User and Password for the monitoring
```

![image](https://user-images.githubusercontent.com/106968663/213886155-5146d9aa-ca2d-4c8a-8957-90c937315ac2.png)

* Now save all the changes and exit the configuration file. To reload the new settings, restart haproxy service.

```
$ sudo systemctl restart haproxy
```

* Next edit the `/etc/hosts` file.

```
$ sudo nano /etc/hosts
```

* Define the hostnames and IP addresses of the haproxy main server and the webservers.

```
10.128.0.25 haproxy
10.128.0.27 web1
10.128.0.27 web2
```

* Save the changes and exit.

### Step 3) Configure Web Servers.

* In this step, we will configure the remaining Linux systems which are the web servers.

* So, log in to each of the web servers `(web-01, web-02)` and install the Nginx web server package using Ubuntu.

* To install nginx, run the following commands:

```
$ sudo apt update
$ sudo apt install nginx
```

* Install the prerequisites:

```
$ sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

* Import an official nginx signing key so apt could verify the packages authenticity. Fetch the key:

```
$ curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
      | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

* Verify that the downloaded file contains the proper key:

```
$ gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

* The output should contain the full fingerprint `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62` as follows:

```
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```

* If the fingerprint is different, remove the file.
* To set up the apt repository for stable nginx packages, run the following command:

```
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

* If you would like to use mainline nginx packages, run the following command instead:

```
$ echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

* Set up repository pinning to prefer our packages over distribution-provided ones:

```
$ echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

* Next, modify the index.html files for each web server.

### For Web Server 1.

* Switch to the root user on your `Web-01`.

```
$ sudo su
```

* Then run the following command for HTTP header response.

```
# echo "<H1>Hello! This is webserver1: LB-IP-10.128.0.27 </H1>" > /var/www/html/index.html
```

### For Web Server 2.

* Similarly, switch to the root user in `Web-02`.

```
$ sudo su
```

* And create the index.html file as shown.

```
# echo "<H1>Hello! This is webserver2: LB-IP-10.128.0.26 </H1>" > /var/www/html/index.html
```

* Next, configure the `/etc/hosts` file.

```
$ sudo nano /etc/hosts
```

* Add the HAProxy entry to each node.

```
> LB-IP-10.128.0.26 haproxy
```

* Save the changes and exit the configuration file.

![Adding of haproxy](https://user-images.githubusercontent.com/106968663/213918521-a8e52049-7c11-4641-8b85-7f55f47291fd.png)

* Be sure you can ping the HAProxy server from each of the web server nodes.

```
$ ping haproxy -c 3
```

![add haproxy](https://user-images.githubusercontent.com/106968663/213918711-8cf3385c-1406-4b0c-9a3f-160d9b6d16a5.png)

### Step 4) Test HAProxy Load Balancing.

* Run the following command to finalize your configuration.

```
$ sudo service haproxy restart
$ systemctl status haproxy.service
```

* Up until this point, we have successfully configured our HAProxy and both of the back-end web servers.
* The server will show `/HTTP/1.1 200 Ok`.

![Load Balancer Image with Nginx](https://user-images.githubusercontent.com/106968663/213919020-1bf87207-5557-48e6-899d-11caaf60a0bf.png)


# 📚 Author 🖋️

[Mustapha Aliyu Galadima](https://github.com/MG-Musty/)
