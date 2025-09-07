Self Host and Install Canvas LMS On Your Server
Canvas is a powerful open-source LMS that can be self-hosted on your own server. This is a detailed guide to help you Install Canvas LMS on Ubuntu using the Apache web server and enabling SSL for secure communication.

Caution: The steps below are fairly technical and should be performed by a server admin. The installation requires full access to the server this can be verified with the "sudo su" command. The requirements for the Canvas LMS when running all components on the same server are:

RAM (Memory): 8 GB RAM
Processor: 4 CPU cores with 2.0 GHz or more
Disk Space: 30GB for storage
OS: Ubuntu 22.04 LTS - This is a MUST
Prerequisite Step:
Create a new user on your Linux system as "canvas". This user will have permission to run the Canvas LMS setup. It is recommended that you have a separate Linux user manage your canvas installation. After executing the below command it will ask you to set the password for this new user, make sure you keep the password handy as it will be used to login and confirm certain actions during the installation

sudo adduser canvas; usermod -aG sudo canvas; su - canvas
Copy
Step 1: Create a PostgreSQL user and databases for Canvas
Once you execute this command, it will ask for your server password and then, your canvas PostgreSQL password. Please note the later one as it will be used when we will edit the canvas database config file.

sudo apt-get install wget ca-certificates -y && wget -qO - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/postgresql.asc && echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list && sudo apt-get update; sudo apt-get install postgresql-14; sudo -u postgres createuser canvas --no-createdb --no-superuser --no-createrole --pwprompt; sudo -u postgres createdb canvas_production --owner=canvas; sudo -u postgres createdb canvas_development --owner=canvas; sudo -u postgres createuser $USER; sudo -u postgres psql -c "alter user $USER with superuser" postgres;
Copy
Step 2: Installing Git, Ruby, Node.js, and Yarn
sudo apt-get install git-core; sudo apt-get install software-properties-common; sudo add-apt-repository ppa:instructure/ruby; sudo apt-get update; sudo apt-get install ruby3.3 ruby3.3-dev zlib1g-dev libxml2-dev libsqlite3-dev postgresql libpq-dev libxmlsec1-dev libidn11-dev curl make g++; curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash; source ~/.bashrc; nvm install 18.20; curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version 1.19.1; export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
Copy
Step 3: Cloning and Install Canvas LMS
current_user=$(whoami); new_directory="/var"; cd "$new_directory"; sudo git clone https://github.com/instructure/canvas-lms.git canvas; sudo chown -R "$current_user":"$current_user" "$new_directory"/canvas; cd canvas; git checkout prod; for config in amazon_s3 database delayed_jobs vault_contents domain file_store outgoing_mail security external_migration; do cp config/$config.yml.example config/$config.yml; done
Copy
Step 4: Configuring Database, Outgoing Mail and Domain Settings
Set your Database credentials in this step, keep everything as it is, and just set the password to the value you entered in Step 1;

Note: When you open a file with  nano command then press ctrl + x then Y to save the changes to the file.

In the next steps, we will use this placeholder {your_domain}.  Make sure you replace this with your actual domain name used for Canvas before executing the commands.
 

cp config/database.yml.example config/database.yml; nano config/database.yml;
Copy
Set the dynamic settings correctly for LTI external tool integrations to work properly.

cp config/dynamic_settings.yml.example config/dynamic_settings.yml; nano config/dynamic_settings.yml;
Copy
Make sure you replace development with production at the top in the dynamic_settings.yml file

production:
# tree
Copy
Set your SMTP mail server details for emails to work on your Canvas LMS. See this guide to learn how to get your SMTP details for Gmail.​

cp config/outgoing_mail.yml.example config/outgoing_mail.yml; nano config/outgoing_mail.yml;
Copy
These settings in the outgoing_mail.yml are tested to work. Note these 2 important parameters,
enable_starttls_auto: false
ssl: true
 production:
address: smtp.gmail.com # Your SMTP server address
port: "465" # Port 465 for SSL
enable_starttls_auto: false # Disable TLS
ssl: true # Enable SSL
user_name: "{Your_SMTP_OR_GMAIL_USERNAME}"
password: "{Your_SMTP_OR_GMAIL_PASSWORD}"
authentication: cram_md5 # secure authentication
domain: smtp.gmail.com # Your SMTP server address
outgoing_address: "{YOUR_EMAIL}"
default_name: "{YOUR_FROM_NAME_ON_EMAILS}"
Copy
Set your domain name under Production -> domain. This should be the domain that is pointed to your server IP address. If you are not sure how to do this see this guide as an example.

cp config/domain.yml.example config/domain.yml; nano config/domain.yml;
Copy
Insert a randomized string of at least 20 characters in production -> encryption_key & set your own domain name in production -> lti_iss: '{your_domain}'. Make sure the domain name is properly set as this is required for LTI external tools to work properly on your Canvas.

cp config/security.yml.example config/security.yml; nano config/security.yml;
Copy
Your security.yml should look something like this,

production: &default
# replace this with a random string of at least 20 characters
encryption_key: daedd3a131ddd8988b14f6e4e01039c93cfa0160
lti_iss: '{your_domain}'
Copy
Step 5: Installing Dependencies and Compiling Assets
sudo apt-get install libyaml-dev; sudo apt-get install cmdtest; sudo gem install bundler --version 2.5.10; bundle config set --local path vendor/bundle; bundle install; sudo gem update strscan; sudo gem uninstall stringio; sudo gem install stringio -v 3.1.1; sudo gem uninstall base64; sudo gem install base64 -v 0.2.0; yarn install; mv db/migrate/20210823222355_change_immersive_reader_allowed_on_to_on.rb .; mv db/migrate/20210812210129_add_singleton_column.rb db/migrate/20111111214311_add_singleton_column.rb; yarn gulp rev; RAILS_ENV=production bundle exec rake db:initial_setup; mv 20210823222355_change_immersive_reader_allowed_on_to_on.rb db/migrate/.; RAILS_ENV=production bundle exec rake db:migrate; mkdir -p log tmp/pids public/assets app/stylesheets/brandable_css_brands; touch app/stylesheets/_brandable_variables_defaults_autogenerated.scss Gemfile.lock log/production.log; RAILS_ENV=production bundle exec rake canvas:compile_assets;
Copy
Need Our Help?
CANVAS INSTALLATION SERVICE
Step 6: Installing and Configuring Apache
sudo apt-get install apache2; sudo apt-get install -y dirmngr gnupg apt-transport-https ca-certificates; sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7; sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger $(lsb_release -cs) main > /etc/apt/sources.list.d/passenger.list'; sudo apt-get update; sudo apt-get install -y libapache2-mod-passenger; sudo a2enmod rewrite; sudo a2enmod passenger; sudo a2enmod ssl; sudo nano /etc/apache2/mods-available/passenger.conf; sudo service apache2 restart;
Copy
Now, you have to add your Linux server user to the passenger.confconfig file. Make sure you add every directive on a new line like this:

PassengerDefaultUser canvas
PassengerStartTimeout 180
PassengerPreloadBundler On
PassengerFriendlyErrorPages On
Step 7: Obtain SSL Certificate For Your Domain
Canvas requires a valid & verified SSL certificate to be installed for your domain, please note that self signed cert will not work. We will use Lets Encrypt to get a free certificate and make it auto-renew so it does not expire after the 3-month period.

A) Install Certbot:

sudo apt update; sudo apt install certbot;
Copy
B) Install certbot plugin for Apache:

# Install Certbot for Apache
sudo apt install python3-certbot-apache
# Install Certbot for Nginx
#sudo apt install certbot python3-certbot-nginx
Copy
C) Obtain the SSL Certificate: Once you execute the below command and follow the steps a SSL cert and private key will be generated on your server. Note the following 2 paths to enter in the next Step 8: Cert: /etc/letsencrypt/live/{your_domain}/fullchain.pem Key: /etc/letsencrypt/live/{your_domain}/privkey.pem

# For Apache
sudo certbot --apache -d {your_domain}
# For Nginx
sudo certbot --nginx
Copy
D) Automatic Renewal: Let's Encrypt SSL certificates are typically valid for 90 days. To automatically renew them, you can set up a cron job in the crontab as follows:

crontab -e
Copy
Add the below line once you've opened the crontab with the above command, save and exit.

0 0 * * * /usr/bin/certbot renew --quiet
Copy
Step 8: Configuring Virtual Hosts for Canvas
First, disable any Apache VirtualHosts you don't want running

sudo unlink /etc/apache2/sites-available/000-default.confs; sudo unlink /etc/apache2/sites-available/default-ssl.conf; sudo unlink /etc/apache2/sites-available/000-default-le-ssl.conf;
Copy
Now we will create a Virtual host for our Canvas LMS Installation.

sudo nano /etc/apache2/sites-available/canvas.conf
Copy
Add the following configuration to canvas.conf:

<VirtualHost *:80>
ServerName {your_domain}
DocumentRoot /var/canvas/public
PassengerRuby /usr/bin/ruby
PassengerAppEnv production
RailsEnv production
<Directory /var/canvas/public>
AllowOverride all
Options -MultiViews
Require all granted
</Directory>
</VirtualHost>
Copy
sudo nano /etc/apache2/sites-available/canvas-ssl.conf
Copy
Add the following configuration to canvas-ssl.conf: Set the paths for these 2 variables according to the value recived in Step 7: for cert and key

SSLCertificateFile and  SSLCertificateKeyFile

<IfModule mod_ssl.c>
<VirtualHost *:443>
ServerName {your_domain}
DocumentRoot /var/canvas/public
PassengerRuby /usr/bin/ruby
PassengerAppEnv production
RailsEnv production
SSLEngine On
SSLCertificateFile /etc/letsencrypt/live/{your_domain}/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/{your_domain}/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf
<Directory /var/canvas/public>
AllowOverride all
Options -MultiViews
Require all granted
</Directory>
XSendFile On
XSendFilePath /var/canvas
</VirtualHost>
</IfModule>
Copy
sudo a2ensite canvas.conf; sudo a2ensite canvas-ssl.conf;
Copy
Step 9: Setup Automated jobs & Firewall Rules
sudo ln -s /var/canvas/script/canvas_init /etc/init.d/canvas_init; sudo update-rc.d canvas_init defaults; sudo /etc/init.d/canvas_init start; sudo ufw allow 80; sudo ufw allow 80/tcp; sudo ufw allow 443; sudo ufw allow 443/tcp; sudo ufw allow 5432; sudo ufw allow 5432/tcp; sudo ufw allow 3001; sudo ufw allow 3001/tcp; sudo ufw allow 3000; sudo ufw allow 3000/tcp; sudo ufw allow 6379; sudo ufw allow 6379/tcp; sudo ufw allow 8000; sudo ufw allow 8000/tcp; sudo ufw allow ssh; sudo ufw enable; sudo ufw reload;
Copy
Step 10: Setup Redis in Cache Configuration
Some of the features of Canvas require Redis, such as OAuth2 which is needed for LTI external tools, so it's required that you setup Redis for caching.

Required version: redis 2.6.x or above.

sudo add-apt-repository ppa:chris-lea/redis-server; sudo apt-get update; sudo apt-get install redis-server; sudo systemctl start redis-server; sudo systemctl enable redis-server; sudo cp config/cache_store.yml.example config/cache_store.yml; sudo nano config/cache_store.yml;
Copy
Make sure the cache_store.yml file contains:

test:
cache_store: redis_cache_store
development:
cache_store: redis_cache_store
production:
cache_store: redis_cache_store
Copy
sudo cp config/redis.yml.example config/redis.yml; sudo nano config/redis.yml;
Copy
Make sure the redis.yml file contains:

production:
url:
- redis://localhost
Copy
sudo systemctl restart redis-server;
Copy
Step 11: Enable Canvas Rich Content Editor
Canvas requires the Canvas RCE API library to be running and configured for full rich content editing functionality. To configure RCE API you will need to get a Flickr & YouTube API Key. Below are the steps to get your keys.

Getting a Flickr API Key:

Create a Flickr Account: If you don't already have one, create a Flickr account at https://www.flickr.com/signup.
Flickr API Key:
Go to the Flickr App Garden: https://www.flickr.com/services/apps/create/apply.
Sign in with your Flickr account.
Fill out the required information, including the name of your app and a short description to get your API Key. Install Canvas LMS flicker-api-key
Getting a YouTube API Key:

Create a Google Account: If you don't have one, create a Google account at https://accounts.google.com/signup.
Google Cloud Console:
Go to the Google Cloud Console: https://console.cloud.google.com/.
Sign in with your Google account.
Create a Project:
Click on the project drop-down menu at the top of the page.
Click "New Project."
Enter a project name and select an organization (if applicable).
Enable the YouTube Data API:
In the left navigation pane, click on "APIs & Services" > "Library."
Search for "YouTube Data API" and select it.
Click the "Enable" button.
Create Credentials:
In the left navigation pane, click on "APIs & Services" > "Credentials."
Click the "Create Credentials" button and select "API Key."
Get Your API Key:
A pop-up will appear with your API key.
You can restrict the API key to prevent unauthorized use if needed. Install Canvas LMS youtube-api-key
git clone https://github.com/instructure/canvas-rce-api.git; cd canvas-rce-api; npm install --production; npm audit fix; cp .env.example .env; ECOSYSTEM_SECRET=$(unique_string=$(head -c 32 /dev/urandom | base64 | tr -d '+/=' | tr -dc 'a-zA-Z0-9' | head -c 32); echo $unique_string); echo $ECOSYSTEM_SECRET; ECOSYSTEM_KEY=$(unique_string=$(head -c 32 /dev/urandom | base64 | tr -d '+/=' | tr -dc 'a-zA-Z0-9' | head -c 32); echo $unique_string); echo $ECOSYSTEM_KEY; CIPHER_PASSWORD=$(openssl rand -hex 16); sed -i "s/^\(NODE_ENV=\).*/\1production/; s/^\(ECOSYSTEM_SECRET=\).*/\1$ECOSYSTEM_SECRET/; s/^\(ECOSYSTEM_KEY=\).*/\1$ECOSYSTEM_KEY/; s/^\(CIPHER_PASSWORD=\).*/\1$CIPHER_PASSWORD/" .env; nano .env
Copy
Add the Flicker and YouTube API keys created previously in the .env file. Copy ECOSYSTEM_KEY & ECOSYSTEM_SECRET from the same file. These 2 values will be used next.

cd ..; cp config/vault_contents.yml.example config/vault_contents.yml; nano config/vault_contents.yml;
Copy
Add the values of ECOSYSTEM_KEY & ECOSYSTEM_SECRET copied from the .env file. Replace develpoment with production in vault_contents.yml.The contents should look like this,

production:
'sts/testaccount/sts/canvas-shards-lookupper-dev':
access_key: 'fake-access-key'
secret_key: 'fake-secret-key'
security_token: 'fake-security-token'
'sts/testaccount/sts/canvas-release-notes':
access_key: 'fake-access-key'
secret_key: 'fake-secret-key'
security_token: 'fake-security-token'
'app-canvas/data/secrets':
data:
canvas_security:
encryption_secret: "YOUR_ECOSYSTEM_KEY"
signing_secret: "YOUR_ECOSYSTEM_SECRET"
Copy
Open the dynamic_settings.yml config file as below.

nano config/dynamic_settings.yml;
Copy
Locate this key rich-content-service -> app-host add your domain name in the first value of app-host and comment out the second value as show,

rich-content-service:
# if you're running canvas-rce-api on its own
app-host: "{your_domain}"
# if you're running canvas-rce-api with docker-compose/rce-api.override.yml in .env
# app-host: "http://rce.canvas.docker:3000"
Copy
IMPORTANT: Editing the YML files is a risky business, do not use tabs to indent while editing these files as this may result in parsing errors. Moreover, if the columns are not properly aligned for values then it will throw a parsing error as well. The error could be something like this. Error starting web application Web application Error column unalign error (): did not find expected key while parsing a block mapping at line 13 column 7 (Psych::SyntaxError) tab error (): found character that cannot start any token while scanning for the next token at line 14 column 5 (Psych::SyntaxError)

SOLUTION: To avoid this, make sure not to use tabs while editing and be sure that you follow the formatting exactly as shown in the code samples any extra space can make the YML file syntax invalid and the application will not start. If you encounter any such issue make sure you run your YML file through a formatter like this one that will point out any syntax issues in the file.

Now, we will set up Apache as a reverse proxy for the Canvas RCE API Node App. To ensure that credentials and payloads are encrypted over the wire https as the node app does not directly supports a secure HTTPS connection.

sudo nano /etc/apache2/sites-available/canvas-ssl.conf;
Copy
Add the below 2 lines before the end of </VirtualHost> block,
ProxyPass /api/session http://localhost:3001/api/session
ProxyPassReverse /api/session http://localhost:3001/api/session
</VirtualHost>
Copy
sudo a2enmod proxy_http; sudo service apache2 restart;
Copy
Finally, start the node app as a screen so that even if you close the terminal window the app is running in the background.

cd canvas-rce-api; sudo apt install screen; screen -S canvas-rce-api;
Copy
npm run start;
Copy
If you are facing any issues with the RCE editor even with these settings then please refer to this Github post by one of the users. Following the steps mentioned here for the RCE configuration should resolve any problems that you might face.

https://github.com/instructure/canvas-rce-api/issues/6#issuecomment-631818899
Now, press Ctrl + a and d  to detach from the screen.

The RCE should be up and running after this final step.

Please note that you will have to start the screen process again if your system reboots. If you want to setup the RCE server in a way that it starts automatically on system reboot then you have to setup it as a Linux service. Follow the guide here to set up the RCE Node.js server as a systemd service instead that starts automatically on system startup.

We know it’s complex—let us help!
CANVAS INSTALLATION SERVICE
Step 12: Set Correct Permissions & ensure users can't read private Canvas files
cd /var/canvas; current_user=$(whoami); sudo chown -R "$current_user":"$current_user" .; sudo find config/ -type f -exec chmod 400 {} +;
Copy
Step 13: Optimizing File Downloads (Optional)
you can optimize the downloading of files using the X-Sendfile header (X-Accel-Redirect in nginx). First make sure that apache has mod_xsendfile installed and enabled. For UBUNTU this can be done by following command:

sudo apt-get install libapache2-mod-xsendfile; sudo systemctl reload apache2; nano config/environments/production-local.rb;
Copy
Add the following lines to production-local.rb:

# If you have mod_xsendfile enabled in Apache:
config.action_dispatch.x_sendfile_header = 'X-Sendfile'
# For nginx:
# config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect'
Copy
Conclusion: Canvas LMS Installed
By following this step-by-step guide, you can Install Canvas LMS on your own Ubuntu Server. Please be advised that these configurations need to be handled by a server administrator.

We know it's quite complex to perform the installation so we are here to help with that as well.