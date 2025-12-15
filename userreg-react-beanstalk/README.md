# Frontend Web App – Digital Banking Portal (React on Elastic Beanstalk)

This folder contains the React frontend web application for the digital banking portal.  
It provides the online banking user interface for login, dashboard, accounts, and transactions, and is intended to be deployed to AWS Elastic Beanstalk.

## Prerequisites

- Node.js (LTS version) and npm installed on your machine.
- An AWS account and basic familiarity with AWS.
- AWS CLI and Elastic Beanstalk CLI (`eb`) installed and configured (for deployment).
- The backend API deployed and reachable (for example, `digital-channels-api-env`).

## Configuration (API base URL)

The frontend needs the base URL of the backend API. This is provided via an environment variable at build time.

- `REACT_APP_API_BASE_URL`  
  The base URL of the backend API, for example:
  - `https://digital-channels-api-env.XXXX.elasticbeanstalk.com`
  - or your custom domain pointing to the API.

For local development, this value can be set in a `.env` file (see `.env.example` below).  
For production builds on Elastic Beanstalk, you can either:
- Build locally with the correct `REACT_APP_API_BASE_URL`, then deploy the built app, or
- Configure environment variables in the Beanstalk environment if you use a Node server.

## Local development

1. Clone the repository (if not already):

git clone https://github.com/pintopathrose/aws-digital-banking-portal-opsack-webapp.git

cd aws-digital-banking-portal-opsack-webapp/userreg-react-beanstalk


2. Install dependencies:

npm install

3. Create a `.env` file in this folder (or use your preferred method) and set:

REACT_APP_API_BASE_URL=http://localhost:3000

Adjust the URL if your backend runs elsewhere.

4. Start the React development server:
npm start


5. Open the app in your browser:

- Go to `http://localhost:3000` (or the port shown in the terminal).
- Login, dashboard, accounts, and transactions should load using your local or remote API.

If your dev port is different, adjust accordingly.

## Deploying to AWS Elastic Beanstalk

There are different ways to host a React app. For this project, the app is deployed as a Node.js application on Elastic Beanstalk that serves the built React files.

### 1. Build the production bundle

From this folder (`userreg-react-beanstalk`):
npm run build

This creates an optimized production build in the `build/` directory.

### 2. Prepare a simple Node.js server (if not already present)

Ensure you have a small `server.js` (or similar) that serves the `build/` folder, for example:
const express = require('express');
const path = require('path');

const app = express();
const port = process.env.PORT || 8080;

app.use(express.static(path.join(__dirname, 'build')));

// Fallback to index.html for SPA routes
app.get('*', (req, res) => {
res.sendFile(path.join(__dirname, 'build', 'index.html'));
});

app.listen(port, () => {
console.log(Web app listening on port ${port});
});


Also make sure your `package.json` has a `start` script that runs this server:

"scripts": {
"start": "node server.js",
"build": "react-scripts build"
}


### 3. Initialize Elastic Beanstalk

From this folder:
eb init


- Choose your AWS region.
- Choose the Node.js platform.

### 4. Create an environment (Free Tier friendly)

Create a new environment for the web app:
eb create digital-channels-web-env --instance_type t3.micro


- `digital-channels-web-env` is an example name; you can choose another name.
- `t3.micro` / `t2.micro` is usually Free Tier eligible.

### 5. Configure environment (optional)

If you need to pass `REACT_APP_API_BASE_URL` at runtime (for example, your Node server reads it), set it in the Elastic Beanstalk environment:

1. Go to the AWS console → Elastic Beanstalk → `digital-channels-web-env`.
2. Open **Configuration → Software**.
3. Add an environment property:
   - `REACT_APP_API_BASE_URL = https://your-api-endpoint-here`

### 6. Deploy changes

Whenever you update the frontend:

1. Rebuild:
npm run build


2. Deploy:
eb deploy

After deployment, Elastic Beanstalk will serve your updated React app.

## Main user flows

Once deployed, the web app supports the following flows:

- **Login**  
  Users enter their credentials, which are sent to the backend `/login` endpoint.

- **Dashboard**  
  Shows a summary of accounts and high-level balances, using the `/accounts` and `/transactions` endpoints.

- **Accounts**  
  Lists all accounts and allows the user to click into an account to see more details.

- **Transactions**  
  Displays recent transactions for a selected account.






