# Backend API – Digital Banking Portal (Elastic Beanstalk)

This folder contains the Node.js backend API for the digital banking portal.  
The API exposes endpoints for login, account summary, and transactions, and is intended to run on AWS Elastic Beanstalk with an RDS database (or mock data for local development).

## Prerequisites

- Node.js (LTS version) and npm installed on your machine.
- An AWS account and basic familiarity with the AWS Management Console.
- AWS CLI and Elastic Beanstalk CLI (`eb`) installed and configured with your AWS credentials.

## Configuration (environment variables)

The API reads its configuration from environment variables. The main ones are:

- `PORT`  
  Port for the HTTP server (default can be 3000).

- `DB_HOST`  
  Hostname or endpoint of the database (e.g., your RDS endpoint).

- `DB_PORT`  
  Database port (e.g., 3306 for MySQL).

- `DB_USER`  
  Database username.

- `DB_PASSWORD`  
  Database password.

- `DB_NAME`  
  Database name.

For local development you can set these in a `.env` file (if your app uses `dotenv`), or export them in your terminal.  
For Elastic Beanstalk, you set them in the environment configuration in the AWS console.

## Local development

1. Clone the repository (if you have not already):

git clone https://github.com/pintopathrose/aws-digital-banking-portal-opsack-webapp.git
cd aws-digital-banking-portal-opsack-webapp/userreg-api-node-beanstalk

2. Install dependencies:
npm install

3. Set environment variables for your local database (or use a local DB / mock config).

4. Start the API locally:
npm start

5. Test the API:

- Health check: `GET http://localhost:3000/health`  
- Login: `POST http://localhost:3000/login`  
- Accounts: `GET http://localhost:3000/accounts`  
- Transactions: `GET http://localhost:3000/transactions`
Adjust the port and paths if your code is different.
## Deploying to AWS Elastic Beanstalk

These steps show how to deploy the backend API to a new Elastic Beanstalk environment.

### 1. Initialize Elastic Beanstalk

From this folder (`userreg-api-node-beanstalk`):
eb init

- Choose:
  - The AWS region you want to use.
  - The platform (for example: Node.js).
- When asked, you can select “Yes” to create an `aws-elasticbeanstalk` configuration folder.

### 2. Create an environment (Free Tier friendly)

Create a new environment with a small instance type:
eb create digital-channels-api-env --instance_type t3.micro


- `digital-channels-api-env` is an example name; you can use a different one if you prefer.
- `t3.micro` (or `t2.micro`) is usually Free Tier eligible in many regions.  
- The first creation may take a few minutes.

### 3. Configure environment variables on AWS

After the environment is created:

1. Open the AWS console.
2. Go to **Elastic Beanstalk → Environments → digital-channels-api-env**.
3. Click **Configuration → Software**.
4. In the “Environment properties” section, add:

   - `DB_HOST` = your RDS endpoint
   - `DB_PORT` = `3306` (or your DB port)
   - `DB_USER` = your DB username
   - `DB_PASSWORD` = your DB password
   - `DB_NAME` = your database name

5. Save the configuration so the environment restarts with these values.

### 4. Deploy changes

Whenever you change the code:
eb deploy


Elastic Beanstalk will build and deploy the new version to the `digital-channels-api-env` environment.

### 5. Test the deployed API

Once deployment finishes:

- Find the environment URL in the Elastic Beanstalk console (something like `http://digital-channels-api-env.XXXX.elasticbeanstalk.com`).  
- Test:

  - `GET <env-url>/health`  
  - `POST <env-url>/login`  
  - `GET <env-url>/accounts`  
  - `GET <env-url>/transactions`
## Logging and health checks

- The API exposes a `GET /health` endpoint that returns a simple JSON status.  
  This can be used by the Application Load Balancer and CloudWatch canaries.

- Each request is logged with:
  - HTTP method and path
  - Response status code
  - A generated request ID

When the app runs on Elastic Beanstalk, these logs are sent to CloudWatch Logs, where they can be used for incident investigation and dashboards.

Adjust this if your code does something slightly different.



