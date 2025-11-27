# -------------------------------
# STAGE 1: Build Stage
# -------------------------------
# Use a lightweight Node.js 20 image based on Alpine Linux for building

FROM node:20.19.5-alpine3.21 AS build

# Set working directory inside the container

WORKDIR /opt/server

# Copy package.json first so Docker layer caching can optimize npm install

COPY package.json .

# Copy all JavaScript files (server.js and others) to the build context
# NOTE: This may copy unnecessary JS files → could impact caching

COPY *.js .

# Install all dependencies listed in package.json

RUN npm install


# -------------------------------
# STAGE 2: Runtime Stage
# -------------------------------
# Use a clean, small Node.js runtime image (Alpine) for running the application

FROM node:20.19.5-alpine3.21

# Set working directory for the application

WORKDIR /opt/server

# Create a system group 'roboshop' and a user 'roboshop'
# Then make the roboshop user the owner of /opt/server directory

RUN addgroup -S roboshop && adduser -S roboshop -G roboshop && \
    chown -R roboshop:roboshop /opt/server

# Expose port 8080 from container → usually for the Node.js service

EXPOSE 8080

# Add metadata labels for identification and documentation

LABEL com.project="roboshop" \
      component="cart" \
      created_by="venky"

# Set environment variables used inside the Node.js application

ENV REDIS_HOST="redis" \
    CATALOGUE_HOST="catalogue" \
    CATALOGUE_PORT="8080"

# Copy the built files from the build stage into this final image
# --chown ensures the roboshop user owns these files

COPY --from=build --chown=roboshop:roboshop /opt/server /opt/server

# Switch to non-root user for better security

USER roboshop

# Default command to run when container starts

CMD ["server.js"]

# Define entrypoint executable → 'node'
# Final command executed will be: node server.js

ENTRYPOINT ["node"]









# FROM node:20.19.5-alpine3.21 AS build
# WORKDIR /opt/server
# COPY package.json .
# COPY *.js .
# # this may add extra cache memory
# RUN npm install 


# FROM node:20.19.5-alpine3.21
# # Create a group and user
# WORKDIR /opt/server
# RUN addgroup -S roboshop && adduser -S roboshop -G roboshop && \
#     chown -R roboshop:roboshop /opt/server
# EXPOSE 8080
# LABEL com.project="roboshop" \
#       component="cart" \
#       created_by="sivakumar"
# ENV REDIS_HOST="redis" \
#     CATALOGUE_HOST="catalogue" \
#     CATALOGUE_PORT="8080"
# COPY --from=build --chown=roboshop:roboshop /opt/server /opt/server
# USER roboshop
# CMD ["server.js"]
# ENTRYPOINT ["node"]



# FROM node:20
# WORKDIR /opt/server
# EXPOSE 8080
# COPY package.json .
# COPY *.js .
# RUN npm install
# ENV REDIS_HOST="redis" \
#     CATALOGUE_HOST="catalogue" \
#     CATALOGUE_PORT="8080"
# CMD ["node", "server.js"]