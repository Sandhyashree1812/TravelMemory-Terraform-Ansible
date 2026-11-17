Project: Travel Memory – Automated MERN Application Deployment on AWS using Terraform and Ansible


----------------------------------------------------------------------------------- 

What are we trying to achieve?
Imagine a user opens our application in a browser.
1. The user accesses the application using the public IP of the web server.
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

Architecture explanation:



<img width="558" height="482" alt="image" src="https://github.com/user-attachments/assets/50ad389b-f1fc-40f3-a0da-93413948dfe2" />
<img width="539" height="137" alt="image" src="https://github.com/user-attachments/assets/9b6d55e1-106c-4eee-837b-b0d29177f2f2" />

Why this architecture?
Component	Why are we using it?
VPC	Creates an isolated AWS network
Public subnet	Hosts the application server
Private subnet	Protects the database server
Internet Gateway	Enables public subnet internet connectivity
NAT Gateway	Allows private instances to download updates
EC2	Provides virtual machines to run the application and database
Security Groups	Control network access
IAM Role	Gives EC2 controlled AWS permissions
Terraform	Automates infrastructure provisioning
Ansible	Automates server configuration

Important security decision: MongoDB should never be publicly accessible. Port 27017 should only accept connections from the web server's security group.

-------------------------------------- 

3. Tools we will use and why
These are the tools I would choose for this project.
Tool	Purpose	Where we use it
AWS	Cloud platform	Entire deployment
AWS CLI	Authenticate and manage AWS	Local machine
Terraform	Infrastructure as Code	Create VPC, EC2, networking
Ansible	Configuration management	Install and configure software
Ubuntu 24.04	Server operating system	Both EC2 instances
Node.js	Backend runtime	Web EC2
React	Frontend	Web EC2
Express.js	REST API	Web EC2
MongoDB	Database	Private EC2
Nginx	Web server and reverse proxy	Web EC2
Git	Version control	Local machine
GitHub	Store deliverables	Repository
SSH	Secure server access	Local machine to EC2
UFW	Host-level firewall	EC2 instances

Why Terraform and Ansible together?
This is an important interview question.

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
This separation is one of the main learning outcomes of the assignment.

----------------------------------------------------------------- 

4. Project execution – 8 phases
We will use the following execution sequence.
1. Phase 1 – Local setup and repository preparation
   
   AWS CLI, Terraform, Ansible, Git and application inspection.
2. Phase 2 – Terraform project setup
   
   AWS provider, variables, networking and infrastructure code.
3. Phase 3 – AWS infrastructure provisioning
   
   VPC, subnets, gateways, route tables, security groups, IAM and EC2.
4. Phase 4 – Ansible configuration
   
   Inventory, SSH connectivity and reusable playbooks.
5. Phase 5 – Database server configuration
   
   MongoDB installation, authentication, database and network security.
6. Phase 6 – MERN application deployment
   
   Node.js, React build, Express backend, Nginx and PM2.
7. Phase 7 – Testing and security verification
   
   Frontend, backend, database connectivity and security checks.
8. Phase 8 – Documentation and submission
   
   GitHub repository, architecture diagram, report and screenshots.

   -----------------------------------

   

1. Project architecture


































































































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
