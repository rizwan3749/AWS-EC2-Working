# AWS-EC2-Working

# README: Steps to Set Up EC2, Deploy Backend, and Generate SSL

## 1. **Start an EC2 Instance**

- Launch a new EC2 instance on AWS.
- Choose an appropriate instance type and OS (e.g., Ubuntu 20.04).
- Configure security groups to allow necessary ports:
  - **22** for SSH
  - **80** for HTTP
  - **443** for HTTPS
- Go to the instance and select the **Security** option, then **Security Groups**. Edit inbound rules and add your custom TCP port with IPv4 as the IP version.
- Allocate and associate an Elastic IP if needed.

---

## 2. **Connect to the EC2 Instance**

- Access the instance using an SSH client.
- Ensure you have the appropriate private key file to connect.
- **Command Example**:
  ```bash
  ssh -i "YOUR.pem" ubuntu@YOUR-IP.compute.amazonaws.com
  ```

---

## 3. **Clone Your Git Repository**

- Navigate to your repository and copy the HTTPS or SSH URL.
- **Command Example**:
  ```bash
  git clone https://github.com/YOUR-REPOSITORY.git
  ```
- Navigate to the cloned directory:
  ```bash
  cd YOUR-REPOSITORY
  ```

---

## 4. **Update and Prepare the Server**

- Update the package manager and install necessary tools.
- **Command Example**:
  ```bash
  sudo apt update
  sudo apt upgrade -y
  sudo apt install -y build-essential curl nano unzip
  ```

---

## 5. **Install Node.js and npm**

- Install Node.js and npm using the NodeSource repository for the latest version.
- **Command Example**:
  ```bash
  sudo apt install -y nodejs
  sudo apt install npm
  node -v
  npm -v
  ```

---

## 6. **Install PM2**

- PM2 is a process manager for Node.js applications.
- **Command Example**:
  ```bash
  sudo npm install -g pm2
  pm2 --version
  ```

---

## 7. **Install and Configure Nginx**

- Install Nginx and start the service.
- **Command Example**:

  ```bash
  sudo apt install nginx -y
  sudo systemctl start nginx
  sudo systemctl enable nginx
  ```

- Configure Nginx as a reverse proxy:

  1. Open the default site configuration file.

     ```bash
     sudo nano /etc/nginx/sites-available/default
     ```

  2. Replace its contents with:

     ```nginx
     server {
         listen 80;
         server_name YOUR-DOMAIN.com;

         # Redirect HTTP to HTTPS
         return 301 https://$host$request_uri;
     }
     ```

  3. Test and reload the configuration:
     ```bash
     sudo nginx -t
     sudo systemctl restart nginx
     ```

---

## 8. **Install Certbot and Generate SSL Certificate**

- Install Certbot and its Nginx plugin.
- Obtain an SSL certificate for your domain.
- **Command Example**:

  ```bash
  sudo apt install certbot python3-certbot-nginx -y
  sudo certbot --nginx -d YOUR-DOMAIN.com -d www.YOUR-DOMAIN.com
  ```

- Update the Nginx configuration for HTTPS:

  ```nginx
  server {
      listen 443 ssl;
      server_name YOUR-DOMAIN.com;

      # Certbot SSL certificates
      ssl_certificate /etc/letsencrypt/live/YOUR-DOMAIN.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/YOUR-DOMAIN.com/privkey.pem;

      # SSL settings for security best practices
      ssl_protocols TLSv1.2 TLSv1.3;
      ssl_ciphers HIGH:!aNULL:!MD5;

      # Proxy all requests to the Node.js server
      location / {
          proxy_pass http://127.0.0.1:PORT;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'upgrade';
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
      }
  }
  ```

- Verify automatic renewal:
  ```bash
  sudo systemctl status certbot.timer
  ```

---

## 9. **Create and Configure Environment Variables**

- Create a `.env` file in your project directory for environment variables.
- Example:
  ```bash
  nano .env
  ```
- Add your variables:
  ```env
  DB_URI=mongodb+srv://user:password@cluster.mongodb.net/dbname
  JWT_SECRET=your_jwt_secret
  PORT=8000
  ```
- Ensure the `.env` file is included in `.gitignore` to prevent it from being pushed to the repository.

---

## 10. **Run Your Backend Application**

- Navigate to your application directory and install dependencies:
  ```bash
  cd YOUR-REPOSITORY
  npm install
  ```
- Start the application using PM2:
  ```bash
  pm2 start index.js --name "backend-app"
  pm2 save
  pm2 startup
  ```

---

## 11. **Monitor and Troubleshoot**

- Check PM2 logs for your application:
  ```bash
  pm2 logs backend-app
  ```
- Restart the application if needed:
  ```bash
  pm2 restart backend-app
  ```

---

## 12. **Additional Commands**

- **Remove a file:**

  ```bash
  sudo rm -rf FILE-NAME
  ```

- **View active ports:**

  ```bash
  sudo lsof -i -P -n | grep LISTEN
  ```

- **Create a new folder:**

  ```bash
  mkdir new_folder
  ```

- **Upload HTML files to Nginx:**
  1. Navigate to the web server directory:
     ```bash
     cd /var/www
     ```
  2. Copy your HTML files:
     ```bash
     sudo cp -r /home/ubuntu/YOUR-HTML-FILE /var/www/YOUR-HTML-FILE
     ```

---

## 13. **Enable Firewall and Allow Ports**

- Configure the Uncomplicated Firewall (UFW) to allow necessary ports:
  ```bash
  sudo ufw allow 'Nginx Full'
  sudo ufw allow OpenSSH
  sudo ufw enable
  ```

---

## Additional Notes

- **Firewall Configuration**: Ensure your AWS security group settings match the necessary inbound/outbound rules.
- **Environment Variables**: Use `.env` files to manage sensitive credentials securely.
- **Backup and Maintenance**: Regularly back up your server and monitor system performance.
- **Automated Deployment**: Consider setting up a CI/CD pipeline to streamline future updates.
- **Test HTTPS**: Ensure your domain is properly secured by testing it with online SSL validation tools.
