Project: Travel Memory – Automated MERN Application Deployment on AWS using Terraform and Ansible

=================================================================================================  

1. Project architecture

1.1 Overall architecture


<img width="566" height="530" alt="image" src="https://github.com/user-attachments/assets/963b227b-27b6-4550-abd8-ac4474def12b" />


<img width="581" height="481" alt="image" src="https://github.com/user-attachments/assets/bbb59961-21ea-447b-a8b5-99e6b70e2341" />


1.2 Architecture in simple diagram form


                  END USER
                     |
                     |
               Web Browser
                     |
                     | HTTP / HTTPS
                     |
              INTERNET GATEWAY
                     |
                     |
       +------------- AWS VPC -------------+
       |                                   |
       |         10.0.0.0/16               |
       |                                   |
       |   PUBLIC SUBNET                   |
       |   10.0.1.0/24                     |
       |                                   |
       |   +---------------------------+   |
       |   | EC2 - WEB SERVER          |   |
       |   | Ubuntu 24.04               |   |
       |   |                           |   |
       |   | Nginx                     |   |
       |   | React Frontend            |   |
       |   | Node.js + Express         |   |
       |   | PM2                       |   |
       |   +---------------------------+   |
       |                |                  |
       |                | MongoDB          |
       |                | Port 27017       |
       |                |                  |
       |   PRIVATE SUBNET                  |
       |   10.0.2.0/24                     |
       |                |                  |
       |   +---------------------------+   |
       |   | EC2 - DATABASE SERVER     |   |
       |   | Ubuntu 24.04               |   |
       |   |                           |   |
       |   | MongoDB                   |   |
       |   | Database: travelmemory    |   |
       |   +---------------------------+   |
       |                                   |
       |   NAT GATEWAY                     |
       |   Private outbound connectivity   |
       |                                   |
       +-----------------------------------+

          INFRASTRUCTURE AUTOMATION

             Terraform
                 |
                 v
          AWS Infrastructure

             Ansible
                 |
                 v
          Server Configuration
                 |
                 v
          Application Deployment

================================================== 

1.3 How the application works
Let's understand the actual request flow.

imagine a user opens our application in a browser.
1. The user accesses the application, Travelmemory using the public IP of the web server.
2. The React frontend loads.
3. React communicates with the Express backend through HTTP API requests.
4. Express processes those requests.
5. Express connects to MongoDB on a separate EC2 instance in the private subnet.
6. MongoDB stores or retrieves the requested information.
7. The backend returns the response to the frontend.
   
The important DevOps concept here is that infrastructure creation and software configuration are automated separately.
- Terraform creates the infrastructure.
- Ansible configures the infrastructure.
- The application runs on the configured infrastructure.

- ---------------------------------------------------------------- 
AWS architecture:
Using a two-tier EC2 architecture, using one public EC2 instance for the application and one private EC2 instance for MongoDB.
-------------------------------------------------------------------- 

2. Architecture explanation:

Why this architecture?
Component	Why are we using it?
. VPC	Creates an isolated AWS network
. Public subnet	Hosts the application server
. Private subnet	Protects the database server
. Internet Gateway	Enables public subnet internet connectivity
. NAT Gateway	Allows private instances to download updates
. EC2	Provides virtual machines to run the application and database
. Security Groups	Control network access
. IAM Role	Gives EC2 controlled AWS permissions
. Terraform	Automates infrastructure provisioning
. Ansible	Automates server configuration

Important security decision: MongoDB should never be publicly accessible. Port 27017 should only accept connections from the web server's security group.

-------------------------------------- 

3. Tools we will use and why
These are the tools we chose for this project.
Tool	                        Purpose	                              Where we use it
AWS	                  Cloud platform	                              Entire deployment
AWS CLI               	Authenticate and manage AWS	               Local machine
Terraform	            Infrastructure as Code	                     Create VPC, EC2, networking
Ansible	               Configuration management	                  Install and configure software
Ubuntu 24.04	         Server operating system                     	Both EC2 instances
Node.js               	Backend runtime	                           Web EC2
React                  	Frontend	                                    Web EC2
Express.js	            REST API	                                    Web EC2
MongoDB	               Database	                                    Private EC2
Nginx	                  Web server and reverse proxy	               Web EC2
Git	                  Version control	                           Local machine
GitHub	               Store deliverables	                        Repository
SSH	                  Secure server access	                        Local machine to EC2
UFW	                  Host-level firewall	                        EC2 instances

---------------------------------------------------- 

Why Terraform and Ansible together?
Terraform answers: What infrastructure should exist?

For example:
- Create a VPC.
- Create subnets.
- Create security groups.
- Create EC2 instances.
- Create an IAM role.
- Allocate a NAT Gateway.

Ansible answers: What should be installed and configured on that infrastructure?

For example:
- Install Node.js.
- Install MongoDB.
- Clone GitHub repository.
- Install NPM dependencies.
- Configure environment variables.
- Start application services.
- 
This separation is one of the main learning outcomes of the assignment.

-----------------------------------------------------------------  


4. Project phases 
We will divide the assignment into these eight phases.

Project execution plan
8 phases

1. Phase 1 – Local environment and project setup
   Starting now
   - Understand the application repository.
   - Prepare the local working directory.
   - Verify AWS CLI.
   - Verify Terraform.
   - Verify Ansible.
   - Configure AWS authentication.
2. Phase 2 – Terraform configuration
   - Initialize Terraform.
   - Configure AWS provider.
   - Create variables and resource files.
   - Prepare infrastructure configuration.
3. Phase 3 – AWS infrastructure provisioning
   - Create VPC.
   - Create public and private subnets.
   - Create Internet Gateway and NAT Gateway.
   - Create route tables.
   - Create security groups.
   - Create IAM roles.
   - Launch two EC2 instances.
4. Phase 4 – Ansible configuration
   - Configure inventory.
   - Configure SSH.
   - Establish web and database connectivity.
   - Test Ansible ping.
5. Phase 5 – MongoDB deployment
   - Install MongoDB.
   - Configure database networking.
   - Enable authentication.
   - Create database and database user.
6. Phase 6 – MERN application deployment
   - Install Node.js and NPM.
   - Clone TravelMemory.
   - Install backend dependencies.
   - Configure environment variables.
   - Build React frontend.
   - Configure Nginx.
   - Start backend with PM2.
7. Phase 7 – Testing and security
   - Test frontend.
   - Test backend APIs.
   - Test MongoDB connectivity.
   - Verify network security.
   - Verify application persistence.
8. Phase 8 – Documentation and submission
   - Architecture documentation.
   - Terraform and Ansible code.
   - Screenshots.
   - README.
   - Implementation report.
   - GitHub submission.


   -----------------------------------
=========================================================================== 

4. Phase 1 – Local environment and project setup

Objective of Phase 1
Before creating AWS infrastructure, we need to prepare our local development environment.

Our local machine will be the Terraform workstation and Ansible control node.

We will use your existing setup:
- Windows laptop
- WSL2 – Ubuntu 24.04
- VS Code
- PowerShell / Ubuntu terminal
- AWS account
- GitHub
We do not need Docker for this assignment. We will deploy directly on EC2 using Terraform and Ansible.

========================================================================= 

Step 1.1 – Understand the TravelMemory application
Before deploying an application, a DevOps Engineer should inspect the source code.

The repository has this structure: 

TravelMemory/
│
├── backend/
│   ├── index.js
│   ├── package.json
│   └── .env.example
│
├── frontend/
│   ├── package.json
│   ├── public/
│   └── src/
│
├── README.md
├── .gitignore
└── azure-pipelines.yml

============================ 

Let's understand what each directory does.
File/folder	                                          Purpose
backend	                                             Express and Node.js application
backend/index.js	                                    Backend application entry point
backend/package.json	                                 Backend dependencies and scripts
frontend	                                             React application
frontend/src	                                       React components and source code
frontend/package.json	                              Frontend dependencies
README.md	                                          Application instructions

Important application details we have identified:
Configuration	                                       Value
Backend port	                                       3001
Backend database variable	                           MONGO_URI
Frontend backend URL variable	                        REACT_APP_BACKEND_URL
Backend health endpoint	                              /health
Travel memories API	                                 /trip
Image uploads	                                       /uploads/

These details will help us configure Ansible correctly later. 

========================================================== 


Step 1.2 – Open Ubuntu WSL
Open VS Code.

Then open your Ubuntu terminal.

You can also open Ubuntu directly from Windows PowerShell:

Run:
wsl
lsb_release -a
pwd
cd ~
pwd

<img width="571" height="303" alt="image" src="https://github.com/user-attachments/assets/a0b43e4c-7673-4e10-9d42-78f851d75bcc" />

---------------------------------------  

Step 1.3 – Verify the required tools
We need five essential tools.

Run these commands one by one:

1. Git
  Run:
git --version
Purpose: Clone the GitHub repository and manage project changes.

3. AWS CLI
aws --version
Purpose: Connect our local machine to AWS and verify our AWS identity.

3. Terraform
terraform --version
Purpose: Create AWS infrastructure using configuration files.

4. Ansible
ansible --version
Purpose: Configure the EC2 instances.

5. SSH
ssh -V
Purpose: Connect securely to EC2 instances.

<img width="880" height="341" alt="image" src="https://github.com/user-attachments/assets/adcaeec5-d4d1-4964-8de1-a56167a99692" />

---------------------------------- 

2. Step 1 – Install Terraform
We will install Terraform using HashiCorp's official Ubuntu package repository.

Why? Terraform is the main tool we will use to create our AWS infrastructure.

Execute these commands in Ubuntu WSL, one at a time.

1. Update package information
   Run: sudo apt-get update

   <img width="725" height="562" alt="image" src="https://github.com/user-attachments/assets/92630a0c-c343-419b-a83e-7d3e8e03f1cc" />

2. Install prerequisites

We need to repair the package manager before installing Terraform.

Step 1 – Repair dpkg
Run this command in your Ubuntu WSL terminal:
sudo dpkg --configure -a
What does it do?
- Completes previously interrupted package installations.
- Configures packages that were unpacked but not configured.
- Repairs the package manager's pending state.

  <img width="806" height="229" alt="image" src="https://github.com/user-attachments/assets/c19c7cf4-ff7f-4acb-8a23-ef05c2431430" /> 

Step 2 – Update package information
Run: sudo apt-get update

<img width="599" height="134" alt="image" src="https://github.com/user-attachments/assets/d7c8de24-a150-4238-ab37-143b34f14aac" />


Step 3 – Install prerequisites
Run: sudo apt-get install -y gnupg software-properties-common wget

<img width="810" height="361" alt="image" src="https://github.com/user-attachments/assets/2485d7b2-5b36-4086-abf2-f67234a5d7ce" />




Step 4 – Verify package manager health
Run: sudo apt-get check

<img width="415" height="81" alt="image" src="https://github.com/user-attachments/assets/e8f92967-f498-4029-965d-a63b10233f30" /> 

----------------------------- 

Step 1 – Update Ubuntu packages
Run:
sudo apt-get update 

<img width="674" height="158" alt="image" src="https://github.com/user-attachments/assets/38c383d5-3ccc-4103-a4ff-dd78fb694a82" />

--------------------------------------- 

3. Add HashiCorp GPG key
Copy the below code and run:
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

<img width="880" height="239" alt="image" src="https://github.com/user-attachments/assets/57f1df81-de6b-4a80-bee0-f452e26d8343" />

--------------------------------- 

5. Add the official repository

 Run:  
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(. /etc/os-release && echo "$VERSION_CODENAME") main" | sudo tee /etc/apt/sources.list.d/hashicorp.list


6. Update and install Terraform
   Run:sudo apt-get update

<img width="886" height="236" alt="image" src="https://github.com/user-attachments/assets/0cfe4b98-b62e-43c0-aca2-35b3e2d01ded" />

Run:sudo apt-get install terraform -y

<img width="864" height="337" alt="image" src="https://github.com/user-attachments/assets/36a3add7-9cf7-4d82-bb61-fbd2858b1e4b" />

7. Verify installation
terraform --version

<img width="429" height="81" alt="image" src="https://github.com/user-attachments/assets/f9d753aa-78dc-496f-bbe2-6c57ac314706" />


Phase 1 – Step 2: Verify AWS authentication
Now we need to verify that our local machine can communicate with the correct AWS account.

1. Check AWS configuration
Execute:
Run: aws configure list

<img width="618" height="104" alt="image" src="https://github.com/user-attachments/assets/e131aa46-1399-469c-a6a8-df71ea3afa81" />


3. Verify AWS identity
Execute:
aws sts get-caller-identity

<img width="608" height="95" alt="image" src="https://github.com/user-attachments/assets/981d278e-6c2b-4890-a2cd-3df9ecea9272" />


Run: aws configure get region
<img width="455" height="29" alt="image" src="https://github.com/user-attachments/assets/3837be89-712f-4232-ad82-c9529892f862" />


3. Check your existing project directories
Now let's inspect your home directory.

Execute:
cd ~

ls -la















































































============================================================ 
============================================================
`.env` file to work with the backend after creating a database in mongodb (see `backend/.env.example`):

```
MONGO_URI='ENTER_YOUR_URL'
PORT=3001
# Frontend origins allowed to call the API (comma separated)
CORS_ORIGIN=http://localhost:3000
# Optional
UPLOAD_DIR=uploads
MAX_UPLOAD_MB=5
TRUST_PROXY=0
```

Data format to be added (dates are `YYYY-MM-DD`):

```json
{
    "tripName": "Incredible India",
    "startDateOfJourney": "2022-03-19",
    "endDateOfJourney": "2022-03-27",
    "nameOfHotels":"Hotel Namaste, Backpackers Club",
    "placesVisited":"Delhi, Kolkata, Chennai, Mumbai",
    "totalCost": 800000,
    "tripType": "leisure",
    "experience": "Lorem Ipsum, Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum,Lorem Ipsum, ",
    "image": "https://t3.ftcdn.net/jpg/03/04/85/26/360_F_304852693_nSOn9KvUgafgvZ6wM0CNaULYUa7xXBkA.jpg",
    "shortDescription":"India is a wonderful country with rich culture and good people.",
    "featured": true
}
```


For frontend, you need to create `.env` file and put the following content (remember to change it based on your requirements):
```bash
REACT_APP_BACKEND_URL=http://localhost:3001
```

## How to run BE

Note: Make sure you have the .env file already added
```bash
cd backend
npm install
npm start        # or: npm run dev (auto-reload with nodemon)
```

## How to run FE
Note: Make sure you have the .env file already added
```bash
cd frontend
npm install
npm start
```

## API

| Method | Path          | Description |
|--------|---------------|-------------|
| GET    | `/trip`       | List trips (featured first, newest first). Optional `?featured=true\|false`. The `experience` text is left out of the list. |
| GET    | `/trip/:id`   | Get one trip. `400` for a malformed id, `404` if it does not exist. |
| POST   | `/trip`       | Create a trip. Send JSON, or `multipart/form-data` with an optional `image` file. Returns `201` with `{ message, trip }`. |
| GET    | `/uploads/:file` | Uploaded images. |
| GET    | `/health`     | `200` when the API and database are up, `503` otherwise. |

Errors are always JSON: `{ "error": "message", "details": { "field": "reason" } }`.

Example upload:

```bash
curl -F tripName="Kyoto" -F startDateOfJourney=2024-11-02 -F endDateOfJourney=2024-11-09 \
     -F shortDescription="Temples and maple leaves" -F tripType=backpacking \
     -F image=@photo.jpg http://localhost:3001/trip
```

### Upload rules
- JPEG, PNG, GIF or WEBP only (SVG is rejected). The real file signature is checked, not only the extension/MIME type.
- Max size `MAX_UPLOAD_MB` (default 5 MB), one file per request.
- Files get a random name and are served with `X-Content-Type-Options: nosniff`.

### Security measures
- `helmet` security headers, CORS limited to `CORS_ORIGIN`, rate limiting (stricter on `POST /trip`), 100kb JSON body limit.
- Input allow-listing and validation (lengths, dates, trip type, cost, http(s)-only image links); NoSQL query sanitising.
- No internal error details are leaked to clients.
- Frontend nginx sends CSP, `X-Frame-Options`, `nosniff` and `Referrer-Policy` headers.

## Docker

```bash
# Backend (uploads are kept in a volume)
docker build -t travelmemory-be ./backend
docker run -p 3001:3001 -e MONGO_URI=... -e CORS_ORIGIN=http://localhost -v tm-uploads:/app/uploads travelmemory-be

# Frontend (the backend URL is baked in at build time)
docker build -t travelmemory-fe --build-arg REACT_APP_BACKEND_URL=http://localhost:3001 ./frontend
docker run -p 80:80 travelmemory-fe
```
