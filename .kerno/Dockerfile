# Multi-stage Dockerfile for NestJS Realworld Example App

# Stage 1: Build stage
FROM node:14-alpine AS builder

WORKDIR /app

# Install build dependencies for native modules (argon2)
RUN apk add --no-cache python3 make g++

# Copy package files
COPY package*.json ./

# Install all dependencies (including devDependencies for build)
RUN npm ci

# Copy source code
COPY . .

# Create the missing config file with SECRET constant
RUN mkdir -p src && \
    echo "export const SECRET = process.env.SECRET || 'secret';" > src/config.ts

# Build the application (compile TypeScript to dist/)
RUN npm run prestart:prod

# Stage 2: Production stage
FROM node:14-alpine

WORKDIR /app

# Install build dependencies for native modules (argon2)
RUN apk add --no-cache python3 make g++

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Remove build dependencies to reduce image size
RUN apk del python3 make g++

# Copy compiled application from builder stage
COPY --from=builder /app/dist ./dist

# Copy index.js (main entry point)
COPY index.js ./

# Expose application port
EXPOSE 3000

# Set NODE_ENV to production
ENV NODE_ENV=production

# Run the application in production mode
CMD ["npm", "run", "start:prod"]
