# Based on the latest version as of 2016-06-04
FROM node:6.2.1

# Let's create our app folder and copy our project files
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY . /usr/src/app

# Open the port of our application
EXPOSE 10010

# Start the application
CMD ["node","app.js"]
