# Canvas LMS Ansible Deployment

This Ansible playbook automates the deployment of Canvas LMS on Ubuntu 22.04 servers following the official installation guide.

## Prerequisites

- Ansible installed on your control machine
- Ubuntu 22.04 LTS server with root/sudo access
- Domain name pointed to your server IP
- Flickr and YouTube API keys (for Rich Content Editor)

## Quick Start

1. **Clone or download this playbook**

2. **Update the inventory file**
   ```bash
   nano inventory.ini
   ```
   Replace `YOUR_SERVER_IP` with your actual server IP address.

3. **Configure variables**
   ```bash
   nano vars/canvas_vars.yml
   ```
   Update all the variables with your specific configuration:
   - `canvas_domain`: Your domain name
   - `postgres_canvas_password`: Database password
   - `smtp_*`: Email server configuration
   - `flickr_api_key` & `youtube_api_key`: API keys for RCE

4. **Run the playbook**
   ```bash
   ansible-playbook -i inventory.ini canvas-deploy.yml
   ```

## Post-Deployment Steps

### 1. Obtain SSL Certificate
```bash
# SSH into your server
ssh ubuntu@your-server-ip

# Run certbot to get SSL certificate
sudo certbot --apache -d your-domain.com
```

### 2. Start Canvas RCE API
```bash
# SSH into your server and start the RCE API service
ssh ubuntu@your-server-ip
sudo su - canvas
cd /var/canvas-rce-api
screen -S canvas-rce-api
npm run start
# Press Ctrl+A then D to detach from screen
```

### 3. Access Canvas
Open your browser and navigate to `https://your-domain.com` to access Canvas LMS.

## Configuration Details

### Required Variables (vars/canvas_vars.yml)

| Variable | Description | Example |
|----------|-------------|---------|
| `canvas_domain` | Your Canvas domain | `canvas.example.com` |
| `canvas_user_password` | Password for canvas system user | `secure_password123` |
| `postgres_canvas_password` | PostgreSQL canvas user password | `db_password123` |
| `smtp_server` | SMTP server address | `smtp.gmail.com` |
| `smtp_port` | SMTP server port | `465` |
| `smtp_username` | SMTP username | `your-email@gmail.com` |
| `smtp_password` | SMTP password/app password | `your_app_password` |
| `flickr_api_key` | Flickr API key for RCE | `your_flickr_key` |
| `youtube_api_key` | YouTube API key for RCE | `your_youtube_key` |

### Getting API Keys

#### Flickr API Key
1. Create account at https://www.flickr.com/signup
2. Go to https://www.flickr.com/services/apps/create/apply
3. Fill out the form to get your API key

#### YouTube API Key
1. Go to https://console.cloud.google.com/
2. Create a new project
3. Enable YouTube Data API v3
4. Create credentials (API Key)

### Gmail SMTP Setup
1. Enable 2-factor authentication on your Gmail account
2. Generate an app password at https://myaccount.google.com/apppasswords
3. Use the app password in the `smtp_password` variable

## File Structure

```
.
├── canvas-deploy.yml           # Main playbook
├── inventory.ini              # Server inventory
├── vars/
│   └── canvas_vars.yml        # Configuration variables
├── templates/                 # Jinja2 templates for configs
│   ├── database.yml.j2
│   ├── dynamic_settings.yml.j2
│   ├── outgoing_mail.yml.j2
│   ├── domain.yml.j2
│   ├── security.yml.j2
│   ├── cache_store.yml.j2
│   ├── redis.yml.j2
│   ├── vault_contents.yml.j2
│   ├── rce-env.j2
│   ├── passenger.conf.j2
│   ├── canvas.conf.j2
│   ├── canvas-ssl.conf.j2
│   └── production-local.rb.j2
└── README.md                  # This file
```

## What the Playbook Does

1. **System Setup**
   - Creates canvas user with sudo privileges
   - Installs PostgreSQL 14 and creates databases
   - Installs Ruby 3.3, Node.js 18.20, and Yarn
   
2. **Canvas Installation**
   - Clones Canvas LMS repository
   - Installs Ruby and Node.js dependencies
   - Configures database, email, and security settings
   - Compiles Canvas assets
   
3. **Web Server Setup**
   - Installs and configures Apache with Passenger
   - Sets up virtual hosts for HTTP and HTTPS
   - Installs SSL certificate tools
   
4. **Additional Services**
   - Installs and configures Redis for caching
   - Sets up Canvas RCE API for rich content editing
   - Configures firewall rules
   
5. **Optimization**
   - Enables X-Sendfile for efficient file downloads
   - Sets proper file permissions

## Troubleshooting

### Common Issues

1. **SSL Certificate Issues**
   - Ensure your domain is properly pointed to your server
   - Run `sudo certbot --apache -d your-domain.com` manually after deployment

2. **RCE API Not Working**
   - Check if the service is running: `screen -r canvas-rce-api`
   - Verify API keys in `/var/canvas-rce-api/.env`

3. **Email Not Working**
   - Verify SMTP credentials
   - Check if Gmail app password is correct
   - Ensure 2FA is enabled on Gmail account

4. **Canvas Not Loading**
   - Check Apache logs: `sudo tail -f /var/log/apache2/error.log`
   - Verify Passenger is running: `sudo passenger-status`

### Logs Location
- Canvas logs: `/var/canvas/log/production.log`
- Apache logs: `/var/log/apache2/`
- Passenger logs: Usually in Apache error log

## Security Notes

- Change all default passwords in `vars/canvas_vars.yml`
- Keep your API keys secure and don't commit them to version control
- Regularly update Canvas and system packages
- Monitor your server for security updates

## Support

For Canvas-specific issues, refer to:
- Canvas Community: https://community.canvaslms.com/
- Canvas Documentation: https://canvas.instructure.com/doc/
- Original installation guide that this playbook automates

## License

This playbook is provided as-is for educational and deployment purposes. Canvas LMS is subject to its own licensing terms.