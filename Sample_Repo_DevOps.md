## 1. Containerize Your MERN Application

### Dockerize the MERN app:

**Steps Taken:**

Create Dockerfile for both the client (React) and the server (Node.js/Express).
- Ensure the backend API (Node.js) connects to MongoDB correctly.
- Example for Node.js backend Dockerfile:
```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:16

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application
COPY . .

# Expose port 5000
EXPOSE 5000

# Run the app
CMD ["npm", "start"]
```

- Example for React frontend Dockerfile:

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:16 as build

# Set the working directory
WORKDIR /app

# Copy and install dependencies
COPY package.json ./
RUN npm install

# Copy the rest of the app
COPY . .

# Build the React app
RUN npm run build

# Use a lightweight web server to serve the build
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

```

- Build an image:
```bash
docker build -t nodejs-sample-app .
docker run -d -p 3000:3000 nodejs-sample-app
```
