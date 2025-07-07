# 3-Tier DevSecOps Project

This repository contains a simple Node.js API and a React client used for a user management demo. Follow the steps below to get the project running locally.

## Setup

1. Install Node.js (version 18 or later is recommended).
2. Install dependencies for both the API and client:

   ```bash
   cd api && npm install
   cd ../client && npm install
   ```

3. Start the API server:

   ```bash
   cd api
   npm start
   ```

4. In a separate terminal, start the React client:

   ```bash
   cd client
   npm start
   ```

5. Open `http://localhost:3000` in your browser to use the application.

The client now displays an animated banner welcoming you to **DevOps Shack**.

## Kubernetes Manifests

The `k8s` directory contains example YAML manifests for deploying the MySQL database, backend API and frontend client on Kubernetes. Persistent volumes use the `ebs-sc` storage class, MySQL is initialized with the `init.sql` script and sensitive values are managed through Kubernetes secrets.

The `frontend` deployment configures an environment variable `REACT_APP_API` with
the internal URL of the backend service (`http://backend:5000`). Ensure the
frontend image is built with this variable so that API requests are routed to the
correct service when running in the cluster.
