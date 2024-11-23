## 1. Containerize Your MERN Application

`My Resource :`  [Social-Echo](https://github.com/Zetrio16/SocialEcho)

### Dockerize the MERN app:

`Orignal Resource:` [Social-Echo](https://github.com/nz-m/SocialEcho)

**Steps Taken:**

Create Dockerfile for both the client (React) and the server (Node.js/Express).
- Ensure the backend API (Node.js) connects to MongoDB correctly.
- Node.js backend Dockerfile:
```dockerfile
#backend-dockefile

# Use a lightweight Node.js image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application
COPY . .

# Expose the backend port (e.g., 4000)
EXPOSE 4000

# Start the application
CMD ["npm", "start"]

```

- React frontend Dockerfile:

```dockerfile

#frontend-dockefile

# Use a lightweight Node.js image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Expose the port the React app will run on (default 3000)
EXPOSE 3000

# Run the React development server
CMD ["npm", "start"]
```

- Check the docker is running:
```bash
docker login                                
docker ps -a
```

- Build an image:
```bash
docker build -t socialecho-backend ./server
docker build -t socialecho-frontend ./client
```

- Running the docker-compose.yaml file:
```bash
docker-compose up --build
```

The application would be running on port:3000 as specified in the docker-compose.yaml file. 

---
### **2. Setting Up a CI/CD Pipeline**

**Steps Taken:**

Created a `.github/workflows/ci-cd.yml` file:

ci-cd.yml:
```yaml

name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1. Checkout code from the repository
      - name: Checkout code
        uses: actions/checkout@v3

      # 2. Set up Docker Buildx
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      # 3. Cache Docker layers
      - name: Cache Docker layers
        uses: actions/cache@v3
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-

      # 4. Build Frontend Docker image
      - name: Build Frontend Docker image
        run: |
          docker build -f client/Dockerfile -t my-frontend:latest ./client

      # 5. Build Backend Docker image
      - name: Build Backend Docker image
        run: |
          docker build -f server/Dockerfile -t my-backend:latest ./server

      # 6. Push Docker images
      - name: Push Docker images
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker tag my-frontend:latest ruperix16/my-frontend:latest
          docker tag my-backend:latest ruperix16/my-backend:latest
          docker push ruperix16/my-frontend:latest
          docker push ruperix16/my-backend:latest

      # 7. Create Backend .env File
      - name: Create Backend .env File
        run: |
          echo "DATABASE_URL=mongodb://mongo:27017/mydb" >> server/.env
          echo "JWT_SECRET=your-secret-key" >> server/.env

      # 8. Install Docker Compose
      - name: Install Docker Compose
        run: |
          sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
          sudo chmod +x /usr/local/bin/docker-compose
          docker-compose version

      # 9. Deploy with Docker Compose
      - name: Deploy with Docker Compose
        run: |
          docker-compose -f docker-compose.yaml up -d

```

Add this file in your repository. It will we hidden to other developers.

```bash
git add .github/workflows/ci-cd.yml
git commit -m "Update workflow with registry and compose file."
git push origin main
```

### **Outcome:**
I implemented a CI/CD pipeline using GitHub Actions to automate the testing, containerization, and deployment process for a full-stack application.

- The pipeline builds separate Docker images for the frontend and backend.
- These images are pushed to my Docker Hub registry (ruperix16) upon every code push to the main branch.
- The application is then deployed using docker-compose, with services orchestrated for the frontend, backend, and MongoDB database.
- `Note that:` The pipeline runs automatically after each push to the main branch, ensuring continuous integration and delivery.

### **Conclusion:**

By integrating containerization with GitHub Actions, I automated the entire build and deployment process for a full-stack application. This workflow ensures reliable, consistent deployments, reduces manual effort, and simplifies application scalability. It highlights the power of Docker and CI/CD pipelines in modern development practices.
