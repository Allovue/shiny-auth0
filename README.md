# Auth0 + Shiny proxy
This is a nodeJS app that sits in front of our [R Shiny reports app for MNPS](https://github.com/Allovue/mnps-pbb) and proxies requests to it. It allows us to manage auth using Auth0. All users of the MNPS PBB app are added on the Auth0.com admin panel. The user experiences the following flow when they visit the site:

1. User visits mnps-pbb.allovue.com
2. Nginx proxies the request to this shiny-auth0 node app, running on port 3000
3. The shiny-auth0 node app bounces the user to auth0.com where they login. They are checked against a list of users managed through our admin panel on auth0 who have been added to the MNPS-PBB "connection" or database managed on Auth0.com. 
4. A successful login bounces the user to the callback URL which is then proxied via the shiny-auth0 node app to the shiny R app on port 3000.
5. Users should get redirected to http://mnps-pbb.allovue.com/reports/mnps-pbb


# Accessing the EC2 Server

* The server is accessed using the `MNPS PBB.pem` file located in 1Password in the Customer VPN/DB Vault. Copy this file to your local machine.
* SSH into the server with `ssh -i /path/to/MNPS\ PBB.pem ubuntu@mnps-pbb.allovue.com`

# Important Notes and Paths

*  Auth0 Admin information is located with Jason, Jake, and Josh as of January 26th, 2021
*  This app is located at `/home/ubuntu/shiny-auth0` with a systemd service at `/etc/systemd/system/shiny-auth0.service` 
*  The R Shiny App is located at `srv/shiny-server` with a systemd service at `/etc/systemd/system/shiny-server.service`
*  Nginx's config is setup at `/etc/nginx/nginx.conf`

# Commands

* Restarting services: `sudo systemctl restart shiny-auth0`, `sudo systemctl restart shiny-server`, `sudo systemctl restart nginx`

# Initial Setup

This node app needs to be cloned or copied to the same server as the shiny R app. Instructions for setup of a basic shiny R server and setting up this shiny-auth0 proxy can be found here: https://auth0.com/blog/adding-authentication-to-shiny-server/

1. Copy or Clone to the server. Currently it's cloned to ~/home/ubuntu/shiny-auth0
2. Copy the `.env.example` file to `.env` and fill in the `client secret` and `client ID` using information from the Auth0 settings page for the MNPS-PBB app. Change the `cookie_secret` to a new string. Leave the `host`, `shiny_port`, and `port` as is.
3. If the server does not have Node installed, install it and then run `npm install` in the shiny-auth0 directory
4. Copy the included `server-config-files/nginx.conf` in this repo to `/etc/nginx/nginx.conf`. If we are running multiple sites on one box this configuration will need to be moved into a site-specific file under `/etc/nginx/sites-available` and the nginx.conf restored to a version that looks for and runs files symlinked from `sites-available` to `sites-enabled`.
5. Add a systemd service to allow control using the `systemctl` command by copying `server-config-files/shiny-auth0.service` to `/etc/systemd/system/shiny-auth0.service`
6. Enable the service to start on boot: `sudo systemctl enable shiny-auth0`
7. Start the Shiny App: `sudo systemctl start shiny-server`
8. Start this app, the Shiny Auth0 proxy: `sudo systemctl start shiny-auth0`
9. Start or restart nginx: `sudo systemctl restart nginx`

You should be able to visit the root url, login via Auth0, and get redirected to mnps-pbb.allovue.com/reports/mnps-pbb

For further customization you can add the following variables to your `.env` file

```bash
# Auto login if the session exists on Auth0 Server
CHECK_SESSION=true
# When logout is called, log the user out of Auth0 aswell
LOGOUT_AUTH0=true
# When logging out is called, must logout of remote idp aswell
# This will force LOGOUT_AUTH0 to true
LOGOUT_FEDERATED=true
```
