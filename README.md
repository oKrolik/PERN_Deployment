# Introduction

I faced significant challenges deploying my first **[PERN](https://medium.com/@ritapalves/get-started-with-the-pern-stack-an-introduction-and-implementation-guide-e33c55d09994)** (PostgreSQL, Express, React and Node.js) stack on **[Digital Ocean](https://www.digitalocean.com/)** since I had never worked with a droplet before. I had to refer to several guides, videos, and forums. After overcoming these obstacles, I decided to document all the steps that worked for me.

*Thanks to: [https://www.youtube.com/watch?v=Nxw2j1-srVc](https://www.youtube.com/watch?v=Nxw2j1-srVc)*

## Project Structure

This is my **initial** project structure. I understand that things **could** be more organized, but it was my **first project** without prior **PERN** knowledge. **Assume** that **all** my **backend requests** go to **/auth** and not **/api**.
```js
.post(`${SERVER_URL}/auth/login`, ...);
```
```js
.get(`${SERVER_URL}/auth/getUserNextEvent`, { ...);
```

### Backend
```
/backend
  |
  |--> /constants
  |       Contains constants according to .env file.
  |
  |--> /controllers
  |       Contains all backend controllers.
  |
  |--> /database
  |       Includes the models, creation, and insertion scripts.
  |
  |--> /middlewares
  |       Holds the authentication middlewares.
  |
  |--> /routes
  |       Defines the routes for GET, POST, UPDATE, DELETE operations.
  |
  |--> /validators
  |       Validates information such as emails.
  |
  |--> .env
  |       Defines the enviroment variables.
  |
  |--> app.js
  |       Starts the backend application.
  |
  |--> config.js
          Sets up and connects to the database.
```


### Frontend/src
```
/frontend/src
  |
  |--> /components
  |       Contains all components such as previews, 
  |       screens, widgets, modals, phases, and questionnaires.
  |
  |--> /constants
  |       Stores text constants for the pages.
  |
  |--> /context
  |       Manages user authentication status.
  |
  |--> /hooks
  |       Handles user authentication and logout logic.
  |
  |--> /pages
  |       Defines the app pages and phases.
  |
  |--> /scripts
  |       Contains utility scripts.
  |
  |--> /styles
          Contains application styling.
```

## Available Scripts

### General Instructions
Before running the scripts, update the variable values in **/frontend/constants/index.js** and **/backend/.env** according to your environment (local or production).

### ~/FRONTEND

#### `npm start`

Runs the app in development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.

#### `npm run build`

Builds the app for production to the **build** folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

*https://facebook.github.io/create-react-app/docs/deployment*

### ~/BACKEND

#### `nodemon start` or `npm start`

Both can run the app in development mode.\
Open [http://localhost:5000](http://localhost:5000) to view it in your browser.

*Nodemon will only work if it's installed and configured in **package.json*****\
*- Installing: `npm i -g nodemon`*\
*- Configuring:*
  ```json
  "scripts": {
      "start": "nodemon my_file.js"
  },
  ```

## Dependencies

### Installation

#### `npm install`

Run in both **/frontend** and **/backend** directories to install all dependencies.

## Database Setup

### Connect to the droplet

#### `~$ ssh root@[ipv4]`

### PostgreSQL Setup

#### Locally:

- Verify status: `pg_lsclusters`

- Start: `sudo pg_ctlcluster <version> main start`

- Restart: `sudo service postgresql restart`

#### Production:

- Verify updates: `sudo apt update`

- Upgrade it: `sudo apt upgrade`

- Install postgres: `sudo apt install postgresql postgresql-contrib`

- Enable postgres: `sudo systemctl enable postgresql`

- Start postgres: `sudo systemctl start postgresql`

#### Acess PostgreSQL

- Open postgres: `sudo -i -u postgres`

- Open DB: `psql`

- Create *project_name* DB: `CREATE DATABASE db_name;`

- Open *project_name* DB: `\c db_name`

#### Troubleshooting Connection Issues

- Verify status: `pg_lsclusters`

- Start: `sudo pg_ctlcluster <version> main start`

- Restart: `sudo service postgresql restart`

- If refused, change PostgreSQL password:
  - Change to postgres user: `sudo -i -u postgres`
  - Open postgres terminal: `psql`
  - Change postgres' user password: `ALTER USER postgres WITH PASSWORD 'new_password';`

- If your're still getting connection refused, update configuration files:
  
  - Find the `postgresql.conf` file, usually at one of:
    - `/etc/postgresql/<version>/main/`
    - `/etc/postgresql/<version>/data/`

  - Edit the `postgresql.conf` file with `nano postgresql.conf` and change the <strong>listen_adresses</strong> line to: 
    ```bash
    listen_addresses = '*'
    ```
  
  - Find the `pg_hba.conf` file, usually at the same directory, edit the `pg_hba.conf` file with `nano pg_hba.conf` and add this line to the <strong>bottom</strong>: 
    ```bash
    host   all     all     0.0.0.0/0   md5
    ```

- Restart the PostgreSQL service: `sudo systemctl restart postgresql`
- Test connection: `psql -h <db_address> -U postgres -d <db_name>`

*Still refused? Google it, solve it and **PR** (Pull Request) this [README](https://github.com/oKrolik/PERN_Deployment)*\
*https://github.com/oKrolik/PERN_Deployment*

## Client Setup

### Connect to the droplet

#### `~$ ssh root@[ipv4]`

### Quick installation:

- Remove the current default html from the server: `rm -rf /var/www/html`

- Create your own: `mkdir /var/www/project_name`

- Install Nginx: `apt install nginx`

- Install UFW (Uncomplicated Firewall): `apt install ufw`

- Enable Nginx's and SSH's UFW: `ufw enable`, `ufw allow "Nginx Full"` and `ufw allow ssh`

- Remove the current default Nginx configuration file: `rm /etc/nginx/sites-available/default` and `rm /etc/nginx/sites-enabled/default`

- Create a new configuration file: `nano /etc/nginx/sites-available/project_name`\
With this inital content:
  
  ```
  server {
    listen 80;

    server_name project_address;

    location / {
          root /var/www/project_name;
          index  index.html index.htm;
          try_files $uri $uri/ /index.html;
    }
    location /auth {
          proxy_pass http://project_address:5000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'upgrade';
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
    }
  }
  ```
    *"**proxy_pass http://project_address:5000;**" -> Remember that I chose to place my backend requests on the /auth endpoint and my port is 5000*

- Create a **.html** file in your new **/var/www/project_name** directory and write anything: `nano /var/www/project_name/index.html` 

- Link the sites-available file to the sites-enabled file: `ln -s /etc/nginx/sites-available/project_name /etc/nginx/sites-enabled/project_name`

Now you can go to <http://project_address/> (FE) and <http://project_address/auth/> (BE).

### TIP #1

- Install `pm2` to prevent killing processes: `npm install -g pm2`

- Create backend instance with: `pm2 start --name backend app.js`

- Let it run alongside with ubuntu: `pm2 startup ubuntu`\
  Now even if you exit the terminal, the process will still run.

```bash
$ pm2 restart app_name
$ pm2 reload app_name
$ pm2 stop app_name
$ pm2 delete app_name
```

*For more commands, refer to the [pm2 Cheat Sheet](https://pm2.keymetrics.io/docs/usage/quick-start/)*

## React App Deployment

- Move to frontend: `cd ~/project_name/frontend`

- Create build file: `npm run build`

- Remove previous file: `rm -rf /var/www/project_name/*`

- Create a new directory to hold our frontend: `mkdir /var/www/project_name/client`

- Make a copy of our build files to the new directory: `cp -r build/* /var/www/project_name/client`

- Reconfigure the path of our /etc/nginx/sites-available/project_name file: `nano /etc/nginx/sites-available/project_name`

- From root **/var/www/project_name;** to **root /var/www/project_name/client;** or just paste this:
  
  ```
  server {
    listen 80;

    server_name project_address;

    location / {
          root /var/www/project_name/client;
          index  index.html index.htm;
          try_files $uri $uri/ /index.html;
    }
    location /auth {
          proxy_pass http://project_address:5000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'upgrade';
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
    }
  }
  ```

- Reload nginx: `systemctl reload nginx`

- Now you can go to [http://project_address/](http://project_address/) (FE) and verify that it worked.

- To make the domain work, update the URLs from address to domain. Ensure your **~/project_name/backend/.env** has:
  
  ```
  POSTGRES_USER="postgres"
  POSTGRES_PASSWORD="************"
  POSTGRES_HOST="db_address"
  POSTGRES_PORT=5432
  POSTGRES_DATABASE="db_name"
  POSTGRES_DIALECT="postgres"

  PORT=5000
  CLIENT_URL="http://project_domain"
  SERVER_URL="http://project_domain"
  ```
- And your **~/project_name/frontend/src/constants/index.js** have:

  ```
  export const PORT = 3000;
  export const CLIENT_URL = "http://project_domain"
  export const SERVER_URL = "http://project_domain";
  ```

- Reconfigure the path in **/etc/nginx/sites-available/project_name**: `nano /etc/nginx/sites-available/project_name`

- From **server_name project_address;** to **server_name project_domain www.project_domain;** or just paste this:

  ```
  server {
    listen 80;

    server_name project_domain www.project_domain;

    location / {
          root /var/www/project_name/client;
          index  index.html index.htm;
          try_files $uri $uri/ /index.html;
    }
    location /auth {
          proxy_pass http://project_address:5000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'upgrade';
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
    }
  }
  ```

- Test nginx: `nginx -t` 

- Reload nginx: `systemctl reload nginx`

- Verify the project domain at <http://project_domain>. If it works, great! If not, repeat the steps from where it was working, you may have missed something, or the domain might not be online yet, as it can take up to 24 hours for the domain provider to communicate with the host.

## Securing with SSL Certification

### Install Certbot

```
apt install certbot python3-certbot-nginx
certbot --nginx -d project_domain -d www.project_domain
```
Follow the prompts to complete the certification.

### Verify and Reload Nginx

```
systemctl status certbot.timer
nginx -t
systemctl reload nginx
```

## React App (re)Deployment - **HTTPS**

Repeat all steps from "React App Deployment" but with **https** instead of **http**.