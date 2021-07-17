FROM node:14

# Create app directory
WORKDIR /app

# Add Source code
COPY . /app

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production


EXPOSE 7100
CMD ["node", "./bin/www"]
