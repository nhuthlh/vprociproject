# 1. Continuous Integration

In this Continuous Integration lab, we build a Jenkins server, a Nexus server and a SonarQube server. The pipeline starts from the source code in GitHub, uses GitHub Webhook to trigger the pipeline, Jenkins use Maven to test, style check and build artifact, integrate SonarQube to run the code analysis and Quality Gate, upload the artifact to Nexus repo, and send Slack notification of the pipeline result.

**The steps to complete the lab:**

1. Access AWS, create the necessary key pair, security groups.
2. Create 3 EC2 instances(Ubuntu) for Jenkins, Nexus and Sonar.
3. Jenkins plugin installation and configuration.
4. Nexus configuration and repository setup.
5. SonarQube setup.
6. Create GitHub repo and migrate code, integrate GitHub repo with VSCode.
7. Build Jenkins jobs with Nexus integration.
8. Setup GitHub WebHook
9. SonarQube integration
10. Upload artifact to Nexus repo
11. Setup Slack notification.

# Installation of Jenkins server

We can use EC2 user data scrip to quickly install Jenkins server.

```
#!/bin/bash
sudo apt update
sudo apt install openjdk-11-jdk -y
sudo apt install maven -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
###
```

Install the necessary Jenkins plugins for the project.

![image](https://user-images.githubusercontent.com/67490369/197383351-81ddbe72-b0ec-4346-98e8-4ac98bc18e16.png)



# Installation of Nexus server

Similarly, we can use EC2 user data script to install Nexus

```
#!/bin/bash
sudo apt update
sudo apt install openjdk-8-jdk -y
sudo mkdir -p /opt/nexus/   
sudo mkdir -p /tmp/nexus/ 
cd /tmp/nexus/
NEXUSURL="https://download.sonatype.com/nexus/3/nexus-3.42.0-01-unix.tar.gz"
wget $NEXUSURL -O nexus.tar.gz
EXTOUT=`tar xzvf nexus.tar.gz`
NEXUSDIR=`echo $EXTOUT | cut -d '/' -f1`
rm -rf /tmp/nexus/nexus.tar.gz
rsync -avzh /tmp/nexus/ /opt/nexus/
sudo useradd -d /opt/nexus -s /bin/bash nexus
sudo chown -R nexus.nexus /opt/nexus 
sudo cat <<EOT> /etc/systemd/system/nexus.service
[Unit]                                                                          
Description=nexus service                                                       
After=network.target                                                            
                                                                  
[Service]                                                                       
Type=forking                                                                    
LimitNOFILE=65536                                                               
ExecStart=/opt/nexus/$NEXUSDIR/bin/nexus start                                  
ExecStop=/opt/nexus/$NEXUSDIR/bin/nexus stop                                    
User=nexus                                                                      
Restart=on-abort                                                                
                                                                  
[Install]                                                                       
WantedBy=multi-user.target                                                      
EOT

echo 'run_as_user="nexus"' > /opt/nexus/$NEXUSDIR/bin/nexus.rc
systemctl daemon-reload
systemctl enable nexus
systemctl start nexus
```

Login to Nexus and create the required repos.


![image](https://user-images.githubusercontent.com/67490369/197383423-330d650b-d1a7-4172-b64f-211e2785f66c.png)


![image](https://user-images.githubusercontent.com/67490369/197383703-5434f871-6250-4d19-a0bc-f3077eb9c431.png)

# Installation of SonarQube server

The user-data script to install SonarQube server

```
#!/bin/bash
cp /etc/sysctl.conf /root/sysctl.conf_backup
cat <<EOT> /etc/sysctl.conf
vm.max_map_count=262144
fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
EOT
cp /etc/security/limits.conf /root/sec_limit.conf_backup
cat <<EOT> /etc/security/limits.conf
sonarqube   -   nofile   65536
sonarqube   -   nproc    409
EOT

sudo apt-get update -y
sudo apt-get install openjdk-11-jdk -y
sudo update-alternatives --config java

java -version

sudo apt update
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
sudo apt install postgresql postgresql-contrib -y
#sudo -u postgres psql -c "SELECT version();"
sudo systemctl enable postgresql.service
sudo systemctl start  postgresql.service
sudo echo "postgres:admin123" | chpasswd
runuser -l postgres -c "createuser sonar"
sudo -i -u postgres psql -c "ALTER USER sonar WITH ENCRYPTED PASSWORD 'admin123';"
sudo -i -u postgres psql -c "CREATE DATABASE sonarqube OWNER sonar;"
sudo -i -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;"
systemctl restart  postgresql
#systemctl status -l   postgresql
netstat -tulpena | grep postgres
sudo mkdir -p /sonarqube/
cd /sonarqube/

sudo curl -O https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.3.0.34182.zip
sudo apt-get install zip -y
sudo unzip -o sonarqube-8.3.0.34182.zip -d /opt/
sudo mv /opt/sonarqube-8.3.0.34182/ /opt/sonarqube

sudo groupadd sonar
sudo useradd -c "SonarQube - User" -d /opt/sonarqube/ -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube/ -R
cp /opt/sonarqube/conf/sonar.properties /root/sonar.properties_backup
cat <<EOT> /opt/sonarqube/conf/sonar.properties
sonar.jdbc.username=sonar
sonar.jdbc.password=admin123
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
sonar.web.host=0.0.0.0
sonar.web.port=9000
sonar.web.javaAdditionalOpts=-server
sonar.search.javaOpts=-Xmx512m -Xms512m -XX:+HeapDumpOnOutOfMemoryError
sonar.log.level=INFO
sonar.path.logs=logs
EOT

cat <<EOT> /etc/systemd/system/sonarqube.service
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
EOT
systemctl daemon-reload
systemctl enable sonarqube.service
#systemctl start sonarqube.service
#systemctl status -l sonarqube.service
apt-get install nginx -y
rm -rf /etc/nginx/sites-enabled/default
rm -rf /etc/nginx/sites-available/default
cat <<EOT> /etc/nginx/sites-available/sonarqube
server{
    listen      80;
    server_name sonarqube.groophy.in;
    access_log  /var/log/nginx/sonar.access.log;
    error_log   /var/log/nginx/sonar.error.log;
    proxy_buffers 16 64k;
    proxy_buffer_size 128k;
    location / {
        proxy_pass  http://127.0.0.1:9000;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;
              
        proxy_set_header    Host            \$host;
        proxy_set_header    X-Real-IP       \$remote_addr;
        proxy_set_header    X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto http;
    }
}
EOT
ln -s /etc/nginx/sites-available/sonarqube /etc/nginx/sites-enabled/sonarqube
systemctl enable nginx.service
#systemctl restart nginx.service
sudo ufw allow 80,9000,9001/tcp
echo "System reboot in 30 sec"
sleep 30
reboot
```

Test login to the Sonar server

![image](https://user-images.githubusercontent.com/67490369/197383766-e1534e9c-ae4d-4769-9c34-124275066c1c.png)


Setup the Jenkins pipeline and run the pipeline

![image](https://user-images.githubusercontent.com/67490369/197404651-23f1f362-cdbf-43b1-af82-498ac34a37dd.png)


Setup GitHub Webhook

![image](https://user-images.githubusercontent.com/67490369/197435522-28d54f94-4bbd-4ae0-9b50-0dda637e63eb.png)



Integrate with SonarQube

![image](https://user-images.githubusercontent.com/67490369/197435555-024d1006-1856-429d-b91d-8e835cda5538.png)



Full CI pipeline is working

![image](https://user-images.githubusercontent.com/67490369/197444933-31574d0d-2989-4eeb-b2e2-dd2c852a1d6c.png)



Artifacts are uploaded to Nexus repo

![image](https://user-images.githubusercontent.com/67490369/197439017-9490c6e3-d84e-4df0-831b-d528d9c026c3.png)


Slack notification upon completion

![image](https://user-images.githubusercontent.com/67490369/197444839-5055160c-ebde-4ec6-b7f7-47add017f120.png)


# 1. Continuous Delivery

In this lab, we continue from the lab above, then we create the AWS ECR and ECS, build the docker image from the code, upload the image to ECR, deploy the image to ECS in Staging enviroment, promote the image to Prod environment.


![image](https://user-images.githubusercontent.com/67490369/197445585-611fccc5-b55b-4241-8efb-8e68bee0f41d.png)


**The steps to complete the lab include:**

1. Prepare 2 Jenkinsfile for staging and prod environment
2. Setup AWS IAM user, ECR repo, ECS clusters for staging and prod.
3. Install the required Jenkins plugins.
4. Install docker engine and aws cli on Jenkins server.
5. Create the Docker file in the GitHub repo
6. Update the Jenkinsfile to include the stages for build and upload image to ECR
7. Add a stage to deploy the Docker image to ECS
8. Update the Jenkinsfile to deploy the approved image to prod ECS cluster.
9. Promote the docker image for prod


Install additional Jenkins plugins required for the lab

![image](https://user-images.githubusercontent.com/67490369/197448307-69b9caf7-6353-4a7b-a271-9628d68d8162.png)

Store AWS access key and secret in Jenkins

![image](https://user-images.githubusercontent.com/67490369/197448751-17ff8216-51bb-46a0-a040-36a592dc8ecc.png)


The pipeline build and publish the docker image to ECR

![image](https://user-images.githubusercontent.com/67490369/197451976-b5de7b5a-2377-41a0-a35e-178df35263c8.png)

The docker image in ECR
![image](https://user-images.githubusercontent.com/67490369/197501442-40a15a36-3108-4c36-8679-2c0a964cc97b.png)


The pipeline deolpy the docker image to ECS

![image](https://user-images.githubusercontent.com/67490369/197500993-bbce4aa7-dcc5-464c-a417-6d86caee5f01.png)


Service deployed to ECS

![image](https://user-images.githubusercontent.com/67490369/197496049-8bb87fd6-9985-48ba-949d-e1d3288b9274.png)


