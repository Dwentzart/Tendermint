### Pointing RPC-API ke subdomain

### RPC-API COSMOS NODE

Set true di dir app.toml `nano .<node>/config/app.toml. set juga ipnya ke 0.0.0.0:port`

#### Update packages
```
sudo apt update && sudo apt upgrade -y
```
#### Install nginx
```
sudo apt install nginx -y
```
#### Install nodejs
```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
```
#### Install yarn
```
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
```
#### API
```
nano /etc/nginx/sites-enabled/<YOUR.API.SUBDOMAIN.SITE>.conf
```
```
server {
    server_name YOUR.API.SUBDOMAIN.SITE;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

        proxy_pass http://YOUR_API_NODE_IP:1337;

    }
}
```
#### RPC 
```
nano /etc/nginx/sites-enabled/<YOUR.RPC.SUBDOMAIN.SITE>.conf
```
```
server {
    server_name YOUR.RPC.SUBDOMAIN.SITE;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

        proxy_pass http://YOUR_RPC_NODE_IP:26657;

    }
}
```
`test config RPC-API`
```
nginx -t 
```
#### Install SSL
```
sudo apt install certbot python3-certbot-nginx
```
```
sudo ufw allow 'Nginx Full'
```
```
sudo certbot --nginx -d <subdomain>
```
### Starting 🌪️
```
sudo systemctl restart nginx
```
