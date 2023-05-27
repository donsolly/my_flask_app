# Use an official Python runtime as a parent image
FROM python:3.8-slim-buster

# Set the working directory in the container to /app
WORKDIR /app

# Add the current directory contents into the container at /app
ADD . /app

# Build having error because of uWSGI==2.0.17.1 (Hotfix)
RUN apt-get update && apt-get install -y --no-install-recommends gcc libc6-dev libpcre3-dev && rm -rf /var/lib/apt/lists/*

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Set the environment variables
ENV MYSQL_SERVICE_PORT=3306

# Make port 5000 available to the world outside this container
EXPOSE 5010

# Run userapi.py when the container launches
CMD ["python", "userapi.py"]
