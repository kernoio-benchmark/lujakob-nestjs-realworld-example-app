# Stage 1: Build stage
FROM node:14-alpine AS builder

# Set working directory
WORKDIR /app

# Install build dependencies for native modules (argon2)
RUN apk add --no-cache python3 make g++

# Copy package files
COPY package*.json ./

# Install all dependencies (including devDependencies for build)
RUN npm ci

# Copy source code
COPY . .

# Create config file with SECRET if it doesn't exist
RUN mkdir -p src && \
    echo "export const SECRET = process.env.SECRET || 'secret';" > src/config.ts

# Build the application
RUN npm run prestart:prod

# Stage 2: Production stage
FROM node:14-alpine

# Set working directory
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

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Copy any other necessary files (like index.js if needed)
COPY index.js ./

# Expose the application port (default NestJS port)
EXPOSE 3000

# Set NODE_ENV to production
ENV NODE_ENV=production

# Set default SECRET (should be overridden in production)
ENV SECRET=changeme

# Run the compiled application directly
CMD ["node", "dist/main.js"]
