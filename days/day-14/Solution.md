To solve this-> you have to check which server has issue.

for that from jumphost call each server.

curl http://stapp01:8087 -> check got result or not
curl http://stapp02:8087
curl http://stapp03:8087


let we get issue in stapp01 server =>

ssh tony@stapp01 -> yes -> Ir0nM@n

# now check apache server enable and started 

sudo systemctl status httpd  

# if disable ->

sudo systemctl enable httpd

# if not started ->

sudo systemctl start httpd

# if get error-> check the port is available or not 

sudo ss -tulpn | grep 8087

-> check port is used by which service -> for example it shows sendmail -> now stop send mail

sudo systemctl stop sendmail -> sudo systemctl start httpd


# now check url ->
curl http://localhost:8087

# if work check form jumphost -> if don't work from jumphost -> enable incoming request from jumphost by iptables / ufw