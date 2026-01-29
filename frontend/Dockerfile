# Stage 1: Build the application
FROM node:20.19.5-alpine AS build

ENV NODE_OPTIONS=--max-old-space-size=4096

# Install dependencies for building native modules
RUN apk add --no-cache python3 make g++

# Set working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to install dependencies
COPY package*.json ./

# Copy local packages (required for file: dependencies)
COPY packages ./packages

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Declare build arguments
ARG VITE_SERVER_URL
ARG VITE_AUTH_URL
ARG VITE_BACKEND_URL
ARG VITE_IAM_URL
ARG VITE_NOTIFICATION_BACKEND_URL
ARG VITE_AI_BACKEND

# Set environment variables for the build
ENV VITE_SERVER_URL=$VITE_SERVER_URL \
    VITE_AUTH_URL=$VITE_AUTH_URL \
    VITE_BACKEND_URL=$VITE_BACKEND_URL \
    VITE_IAM_URL=$VITE_IAM_URL \
    VITE_NOTIFICATION_BACKEND_URL=$VITE_NOTIFICATION_BACKEND_URL \
    VITE_AI_BACKEND=$VITE_AI_BACKEND

# Build the Vite app
RUN npm run build

# Stage 2: Serve the built app
FROM node:20.19.5-alpine

# Install the 'serve' package to serve static files
RUN npm install -g serve

# Set working directory inside the container
WORKDIR /app

# Copy only the built files from the previous stage
COPY --from=build /app/dist ./dist

# Expose port 3001 for serving the app
EXPOSE 3001

# Serve the built application
CMD ["serve", "-s", "dist"]