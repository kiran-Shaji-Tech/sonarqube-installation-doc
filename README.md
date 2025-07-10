# sonarqube-installation-doc
This guide provides step-by-step instructions to install and configure SonarQube on an Ubuntu 22.04 LTS server.

📦 1. **Update Your Server**
SSH into your server as a non-root user with sudo privileges and update the system:
sudo apt update && sudo apt upgrade -y

☕ 2. **Install OpenJDK 17**
sudo apt install -y openjdk-17-jdk

🐘3. **Install and Configure PostgreSQL**

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -


**Install PostgreSQL:**

sudo apt update
sudo apt install -y postgresql postgresql-contrib

**Enable and start the service:**

sudo systemctl enable postgresql
sudo systemctl start postgresql

**Configure PostgreSQL:**

sudo passwd postgres
su - postgres
createuser sonar
psql

**In the PostgreSQL shell:**

ALTER USER sonar WITH ENCRYPTED PASSWORD 'your_strong_password';

CREATE DATABASE sonarqube OWNER sonar;

GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;

\q

Exit the postgres user:

exit

📥 4. **Download and Install SonarQube**

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.7.0.110598.zip

sudo apt install -y zip

sudo unzip sonarqube-25.7.0.110598.zip

sudo mv sonarqube-25.7.0.110598 /opt/sonarqube


👤 5. **Create SonarQube User and Group**

sudo groupadd sonar

sudo useradd -d /opt/sonarqube -g sonar sonar

sudo chown -R sonar:sonar /opt/sonarqube


⚙️ 6. **Configure SonarQube**

Edit SonarQube's configuration:

sudo vi /opt/sonarqube/conf/sonar.properties

Uncomment and update:
  
sonar.jdbc.username=sonar

sonar.jdbc.password=your_strong_password

sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube


**Edit the sonar.sh script:**

sudo vi /opt/sonarqube/bin/linux-x86-64/sonar.sh

Uncomment and modify:

RUN_AS_USER=sonar

You can add this in the botton line of the configuration file.

🛠️ 7. **Create a systemd Service for SonarQube**

Create the service file by,

sudo vi /etc/systemd/system/sonar.service


Paste the following:
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar
Group=sonar
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target

**Enable and start SonarQube:**

sudo systemctl enable sonar
sudo systemctl start sonar
sudo systemctl status sonar

🧠 8. **Update Kernel Limits**

Edit sysctl config:

sudo vi /etc/sysctl.conf

Add:

vm.max_map_count=262144
fs.file-max=65536

Edit /etc/security/limits.conf to add:

* - nofile 65536
* - nproc 4096

🌐 9. **Access the SonarQube Web Interface**

Visit: https://your-server-ip:9000

⚠️ Important Security Notice: SonarQube ships with default admin credentials (admin / admin). For your security, change the password immediately upon first login.

🔒 **Troubleshooting Authentication Issues**

If you experience authentication or session issues when accessing SonarQube, especially after a successful installation:

Please install SSL and configure Nginx as a reverse proxy with a proper domain name. This ensures secure communication and reliable session handling.

