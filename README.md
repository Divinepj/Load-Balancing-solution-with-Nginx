# Load-Balancing-solution-with-Nginx

Load Balances are servers that forward traffic to multiple
servers (e.g., EC2 instances) downstream. this means distributing work or tasks among several servers so that no one server gets overloaded with too much work. this helps to keep everything running smoothly and ensures that websites and apps work quickly and do not get too slow.
![alt text](./IMG/Snipaste_2023-10-29_20-19-03.png)

## Setting up a Basic Load Balancer
For this project we are going to provision two (2) EC2 instances  running on Ubuntu that will serve as the servers and the provision a third EC2 instance on the AWS console that will serve as the load balancer through Nginx ![alt text](./IMG/Snipaste_2023-10-29_18-37-13.png)
On both server EC2 instances we will configure the security group by opening port `8000` ![alt text](./IMG/Snipaste_2023-10-29_18-40-50.png) 
we will now connect to the EC2 server instances via SSH as shown below ![alt text](./IMG/Snipaste_2023-10-29_18-49-01.png) ![alt text](./IMG/Snipaste_2023-10-29_18-49-57.png)

next we will install Apache on the server instances by running `sudo apt update -y &&  sudo apt install apache2 -y` command as shown below ![alt text](./IMG/Snipaste_2023-10-29_18-53-47.png)
 and then verify that apache is running using the `sudo systemctl status apache2` command as shown below ![alt text](./IMG/Snipaste_2023-10-29_18-55-15.png) ![alt text](./IMG/Snipaste_2023-10-29_18-55-54.png)

 ### Configuring Apache to serve a page showing public IP:
 Here we will configure **Apache** webserver to serve content on port 8000 instead of its default which is port 80. We will do this by opening the file by running `sudo vi /etc/apache2/ports.conf` command and update the file by **ADDING** `Listen 8000` as a new listen directive and save the file for both servers as shown below ![alt text](./IMG/Snipaste_2023-10-29_18-58-16.png) ![alt text](./IMG/Snipaste_2023-10-29_18-59-55.png)  

 Next we will open the file on /etc/apache2/sites-available/000-default.conf and change port 80 on the virtualhost to 8000 for both servers by running `sudo vi /etc/apache2/sites-available/000-default.conf` command as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-01-54.png) ![alt text](./IMG/Snipaste_2023-10-29_19-01-14.png)
 we then save the file and close by running `esc :wq`
 We then restart apache to load the configuration using the command for both servers `sudo systemctl restart apache2` as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-03-31.png) ![alt text](./IMG/Snipaste_2023-10-29_19-04-30.png)

 Next we create a new **index.html** file that will contain code to display the public IP of the EC2 instance by running `sudo vi index.html` and inserting the html file below [before pasting the html file we update the public IP of our EC2 instance from AWS management console and replace the placeholder text for IP address in the html file] for both servers
```
{
        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>


}
```
as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-06-53.png) ![alt text](./IMG/Snipaste_2023-10-29_19-09-13.png)

We then change the ownership of the index.html file with `sudo chown www-data:www-data ./index.html` command as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-10-03.png) ![alt text](./IMG/Snipaste_2023-10-29_19-10-43.png)

Finanlly we will override the default html file of Apache web server by running `sudo cp -f ./index.html /var/www/html/index.html` command for both servers as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-11-49.png) ![alt text](./IMG/Snipaste_2023-10-29_19-12-12.png)
We will then restart the web servers by running `sudo systemctl restart apache2` command and open the public IP of our webservers on a browser as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-14-34.png) and ![alt text](./IMG/Snipaste_2023-10-29_19-15-16.png)

### Configuring Nginx as a Load Balancer
On the third EC2 instance (Load Balancer) we will configure the security group by opening port `80` ![alt text](./IMG/Snipaste_2023-10-29_18-52-10.png) 
we will now connect to the LB EC2 server instance via SSH as shown below ![alt text](./IMG/Snipaste_2023-10-29_18-50-36.png)

next we will install Nginx on the LB instance by running `sudo apt update -y && sudo apt install nginx -y` command as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-16-52.png)
 and then verify that Nginx is running using the `sudo systemctl status nginx` command as shown below ![alt text](./IMG/Snipaste_2023-10-29_19-17-34.png)

 Next we will open the Nginx load balancer configuration file by running `sudo vi /etc/nginx/conf.d/loadbalancer.conf` then we paste the configuration file below to configure Nginx to act like a load balancer [before pasting the html file we update the public IP of our server EC2 instance from AWS management console and replace the placeholder text for IP address in the html file] for both Instance


        
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server 127.0.0.1:8000; # public IP and port for webserser 1
            server 127.0.0.1:8000; # public IP and port for webserver 2

        }

        server {
            listen 80;
            server_name <your load balancer's public IP addres>; # provide your load balancers public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
    

as shown below ![alt text](./IMG/Snipaste_2023-10-29_20-09-23.png)

**Upstream backend_servers** defines a group of backend servers. The **server** lines inside the **upstream** block list the adressess and ports of our backend server. **Proxy_pass** inside the **location** block sets up the the load balancing, passing the requests to the backend servers. the **proxy_set_header** lines pass necessary headers to the backend servers to correctly handle the requests.

We then save the configuration and test by running `sudo nginx -t` command as show shown below ![alt text](./IMG/Snipaste_2023-10-29_20-03-36.png)

Finally we restart the Nginx to load the new conguguration by running `sudo systemctl restart nginx` 

In the end when we run the public IP address of the Nginx Load Balancer on our web browser, the web pages served by both server EC2 instances will be served as shown below ![alt text](./IMG/Snipaste_2023-10-29_20-15-29.png) ![alt text](./IMG/Snipaste_2023-10-29_20-15-52.png).
