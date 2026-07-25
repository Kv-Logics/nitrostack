FROM node:20-alpine

WORKDIR /app

# Copy package.json and install dependencies
COPY package.json ./
RUN npm install

# Copy application code
COPY . .

# Build the project (if necessary)
RUN npm run build

# Expose the MCP server port (defaults to 3001 if using Nitrostack HTTP)
EXPOSE 3001

# Start the MCP server
CMD ["npm", "run", "start:prod"]
