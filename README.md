# sonar-testing-for-java-web-app

This guide details the steps to set up SonarQube on an AWS EC2 t3.medium instance for a testing project, using the recommended minimum configuration with the embedded Elasticsearch and H2 database.


### 1. Launch an AWS EC2 t3.medium Instance
Log in to your AWS Management Console.

Navigate to EC2 → Instances → Launch Instance.

Choose Amazon Linux 2023 / Ubuntu 22.04 LTS as the AMI.

Select t3.medium (2 vCPUs, 4 GiB RAM).

Note: This is the minimum recommended for SonarQube testing with embedded Elasticsearch.

Configure instance details: Enable Auto-assign Public IP.

Add Storage:

Set the root volume size to 20–30 GiB (for SonarQube and project artifacts).

Configure Security Group: Open the following ports:

22 → SSH (Your IP only)

9000 → SonarQube Web (Anywhere/Your IP)

9001 → Elasticsearch (Optional, for internal testing/monitoring)

Review and launch the instance with a key pair for SSH access.

### 2. Connect to the Instance via SSH
Replace <your-key>.pem and <EC2_PUBLIC_IP> with your actual values.

Bash
ssh -i <your-key>.pem ubuntu@<EC2_PUBLIC_IP>

<img width="1177" height="236" alt="Image" src="https://github.com/user-attachments/assets/b21a6066-7e57-4259-bea3-0d52a7f50ba5" />

### 3. Update the System and Install Dependencies
Java 17 is required for SonarQube 10.x and Elasticsearch compatibility.

Bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jdk wget unzip curl maven git

<img width="1129" height="336" alt="Image" src="https://github.com/user-attachments/assets/fe684783-4a92-4b78-8420-66f0ef49f484" />

### 4. Create a Dedicated SonarQube User
It's best practice to run SonarQube under a non-root user.

Bash
sudo adduser sonar
sudo usermod -aG sudo sonar


<img width="739" height="166" alt="Image" src="https://github.com/user-attachments/assets/79fb716c-d99c-4b2f-8ab8-42f23c1b46fc" />

### 5. Download and Install SonarQube
Navigate to /opt, download the distribution, and configure ownership and system limits.

Bash
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.zip
sudo unzip sonarqube-10.4.1.zip
sudo mv sonarqube-10.4.1 sonarqube
sudo chown -R sonar:sonar sonarqube

<img width="1800" height="275" alt="Image" src="https://github.com/user-attachments/assets/9e23f45c-e991-4d5e-a0a7-9cf611481c3f" />

### 6. Configure SonarQube
Edit the sonar.properties file:

Bash
sudo nano /opt/sonarqube/conf/sonar.properties
Update these lines to allow access from any IP:

Properties
sonar.web.host=0.0.0.0
sonar.web.port=9000
Note: The default embedded H2 database is used for testing purposes. It is not recommended for production.

<img width="1800" height="1169" alt="Image" src="https://github.com/user-attachments/assets/3fd5b24d-4a76-414d-a330-f673b57aa721" />

Set Java 17 for the Sonar User

For the sonar user environment:

Bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH



### 7. Start SonarQube
Switch to the sonar user and start the service.

Bash
sudo su - sonar
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
sleep 10
./sonar.sh status
If successful, the output should show: SonarQube is running (pid XXXX).

<img width="1148" height="245" alt="Image" src="https://github.com/user-attachments/assets/ea5fbfaa-e888-470f-b4ba-668523ca5d4a" />


Access the Web Interface

Access SonarQube via your browser:

http://<EC2_PUBLIC_IP>:9000
Default Credentials: admin / admin (You will be prompted to change this upon first login).





### 8. Prepare a Sample Project (Maven)
Clone and build a sample Java project.

Bash
git clone <your-java-web-project.git>

<img width="1560" height="178" alt="Image" src="https://github.com/user-attachments/assets/c5326668-2c7b-4708-8df5-e8c7c52bdd86" />

cd JavaWebCal # (Example directory)
Note: Ensure your project's pom.xml is configured for Java 17.

Build the project:

Bash
mvn clean package

<img width="1367" height="577" alt="Image" src="https://github.com/user-attachments/assets/4a9254f1-3cf2-45c6-930a-7a882a7e4112" />


### 9A. Generating a SonarQube Token

SonarQube uses tokens for secure authentication instead of passwords. You need a token to run mvn sonar:sonar or sonar-scanner commands.

Steps to Generate a Token:
Open the SonarQube web interface: http://<your-server-ip>:9000
Log in with your account (admin/admin for testing).
Click on your avatar → My Account → Security.
Under Tokens, click Generate Token.
Enter a descriptive name (e.g., AngularCalculator-Token) and click Generate.
Copy the generated token immediately.
This token will not be shown again.

<img width="1517" height="522" alt="Image" src="https://github.com/user-attachments/assets/4ff445ef-dcd5-4e92-848d-ba6370708890" />


### 9B. Using the Token in Maven
Use the token in your Maven command as the -Dsonar.login parameter:
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=AngularCalculator \
  -Dsonar.projectName="Angular Calculator" \
  -Dsonar.host.url=http://<your-server-ip>:9000 \
  -Dsonar.login=<your_sonarqube_token>
Replace <your_sonarqube_token> with the token you just generated.
The token acts as the authentication mechanism for the SonarQube server.

<img width="729" height="154" alt="Image" src="https://github.com/user-attachments/assets/eec9c22b-21dd-4098-bd3b-f7c8742a1b53" />


### 10. View Project Results in SonarQube Web Interface
Open your browser: http://<EC2_PUBLIC_IP>:9000

Login with the credentials (default: admin / admin).

Navigate to Projects → WebAppCal and click on the project name.

Explore the various tabs:

Dashboard: See overall code quality, code smells, bugs, vulnerabilities.

Measures: View detailed metrics like coverage, duplications, complexity.

Issues: List of code issues detected with priority and type.

You can click any issue to see the file path, line number, and suggested fixes.



<img width="1515" height="736" alt="Image" src="https://github.com/user-attachments/assets/2625bcb7-cf6e-4607-90df-3e381101b859" />



✅ Summary of Setup

Component	Detail
AWS Instance	t3.medium, Ubuntu 22.04
Java	OpenJDK 17
SonarQube Version	10.4.1
Database	Embedded H2 (Testing Only)
SonarQube Access	http://<EC2_PUBLIC_IP>:9000
Default Login	admin / admin
Tool	SonarScanner CLI
