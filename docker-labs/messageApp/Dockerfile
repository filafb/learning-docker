# use node 4.4.5 LTS
FROM node:10
# Copy source code
COPY . /app
# Change working Directory
WORKDIR /app
# Install Dependencies
RUN npm install
# Expose API port to the outside
EXPOSE 80
# Launch application
CMD ["npm", "start"]
