# Continuous Integration

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

![image](https://user-images.githubusercontent.com/67490369/197553213-7c6b65f7-62ea-4ac2-8b9c-29af18836e2b.png)

![image](https://user-images.githubusercontent.com/67490369/197553268-55c667c1-6927-450b-ba13-1ee8fa0529ac.png)



# Installation of Jenkins

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

![image](https://user-images.githubusercontent.com/67490369/197383351-81ddbe72-b0ec-4346-98e8-4ac98bc18e16.png)






NEXUS



![image](https://user-images.githubusercontent.com/67490369/197383423-330d650b-d1a7-4172-b64f-211e2785f66c.png)


![image](https://user-images.githubusercontent.com/67490369/197383703-5434f871-6250-4d19-a0bc-f3077eb9c431.png)

![image](https://user-images.githubusercontent.com/67490369/197383766-e1534e9c-ae4d-4769-9c34-124275066c1c.png)




![image](https://user-images.githubusercontent.com/67490369/197404005-96f17c85-75f1-4447-907a-96c206e9955b.png)


![image](https://user-images.githubusercontent.com/67490369/197404651-23f1f362-cdbf-43b1-af82-498ac34a37dd.png)

![image](https://user-images.githubusercontent.com/67490369/197435522-28d54f94-4bbd-4ae0-9b50-0dda637e63eb.png)

![image](https://user-images.githubusercontent.com/67490369/197435555-024d1006-1856-429d-b91d-8e835cda5538.png)

![image](https://user-images.githubusercontent.com/67490369/197438717-5009256c-4d8c-4071-babf-8af13e7e7229.png)

![image](https://user-images.githubusercontent.com/67490369/197439017-9490c6e3-d84e-4df0-831b-d528d9c026c3.png)

![image](https://user-images.githubusercontent.com/67490369/197444839-5055160c-ebde-4ec6-b7f7-47add017f120.png)



![image](https://user-images.githubusercontent.com/67490369/197444933-31574d0d-2989-4eeb-b2e2-dd2c852a1d6c.png)


![image](https://user-images.githubusercontent.com/67490369/197445585-611fccc5-b55b-4241-8efb-8e68bee0f41d.png)



![image](https://user-images.githubusercontent.com/67490369/197448307-69b9caf7-6353-4a7b-a271-9628d68d8162.png)

![image](https://user-images.githubusercontent.com/67490369/197448751-17ff8216-51bb-46a0-a040-36a592dc8ecc.png)

![image](https://user-images.githubusercontent.com/67490369/197451976-b5de7b5a-2377-41a0-a35e-178df35263c8.png)

![image](https://user-images.githubusercontent.com/67490369/197501442-40a15a36-3108-4c36-8679-2c0a964cc97b.png)


![image](https://user-images.githubusercontent.com/67490369/197451917-ebcb2830-c8e1-439f-b7bc-7cb23e329112.png)


![image](https://user-images.githubusercontent.com/67490369/197496049-8bb87fd6-9985-48ba-949d-e1d3288b9274.png)

![image](https://user-images.githubusercontent.com/67490369/197500993-bbce4aa7-dcc5-464c-a417-6d86caee5f01.png)

