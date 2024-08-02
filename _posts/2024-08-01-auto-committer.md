---
layout: post
title: "Auto-Committer: Automate Your Git Commits with Docker"
subtitle: Regularly commits and pushes changes to a specified Git repository.
cover-img: assets/img/auto-committer/auto-commiter.jpeg
thumbnail-img: assets/img/auto-committer/auto-commiter.jpeg
share-img: assets/img/auto-committer/auto-commiter.jpeg
categories: [GitHub, Docker, Automation, AutoCommits]
tags: [GitHub Actions, Automation, GitHub, AutoCommits]
author: Dhirendra Singh
---

<!-- Introduction -->
The **Auto-Committer** is a Dockerized Python application that automates the process of committing and pushing changes to a Git repository at regular intervals. This tool is designed to keep your repositories up-to-date without manual intervention, ensuring that your codebase remains current with minimal effort.


## Features

- **Automatic Commits:** Regularly commits and pushes changes to a specified Git repository.
- **Configurable Intervals:** Set the interval at which commits are made.
- **Logging:** Tracks commit activities and errors in a log file.
- **Dockerized:** Easily deployable with Docker and Docker Compose.

## Prerequisites
1. Docker
2. Docker Compose
3. Git

## Installation
Clone the Repository:
```bash
git clone https://github.com/hanisntsolo/auto-committer.git
cd auto-committer
```

## Configure Environment Variables:
Create a .env file in the project root directory with the following variables:
```env
AUTO_COMMIT_REPO_PATH=/path/to/your/repository
COMMIT_SPECIFIC_DIRECTORY=/path/to/directory
GITHUB_TOKEN=your_github_token
LAB_REPO_OWNER=your_github_username
LAB_REPO_NAME=your_repository_name
COMMITTER=your_name
USEREMAIL=your_email@example.com
USERNAME=your_username
COMMIT_INTERVAL_SECONDS=600
LAB_LOG_FILE_PATH=/path/to/your/logfile.log
```

## Build and Run the Application:
```bash
./rebuild_and_run.sh
or
sh rebuild_and_run.sh
```

## Usage
The application runs in a Docker container and commits changes to the specified Git repository at the interval defined in the environment variables.
Logs are saved to the file specified by LAB_LOG_FILE_PATH.
Docker Compose Configuration
Build the Docker image:

The docker-compose.yml file defines the service configuration:

```yaml
version: '3.8'

services:
  commiter_app:
    build: .
    restart: always
    container_name: auto-committer
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
    ports:
      - "5009:5009"
    environment:
      - AUTO_COMMIT_REPO_PATH=${AUTO_COMMIT_REPO_PATH} #Directory which needs to setup as auto commit
      - COMMIT_SPECIFIC_DIRECTORY=${COMMIT_SPECIFIC_DIRECTORY} #Directory which needs to setup as auto commit
      - GITHUB_TOKEN=${GITHUB_TOKEN} # your github PAT(Personal Access Token)
      - LAB_REPO_OWNER=${LAB_REPO_OWNER} # usually github username
      - LAB_REPO_NAME=${LAB_REPO_NAME} # repo name
      - LAB_LOG_FILE_PATH=${LAB_LOG_FILE_PATH} #log file path
      - COMMITTER=${COMMITTER} # name of the commiter 
      - USEREMAIL=${USEREMAIL} # github user email
      - USERNAME=${USERNAME} # github username
      - COMMIT_INTERVAL_SECONDS=600 # commit interval
    volumes:
      - ${COMMIT_SPECIFIC_DIRECTORY}:/${COMMIT_SPECIFIC_DIRECTORY} # volume mapping for the docker container to be able to access the repo from inside the container.
```
Dockerfile:

```Dockerfile
# Use the official Python image from the Docker Hub
FROM python:3.9-slim
RUN pip install --upgrade pip
# Install Git
RUN apt-get update && \
    apt-get install -y git
# Set the working directory
WORKDIR /

# Copy the requirements file into the container
COPY requirements.txt requirements.txt

# Install the dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application code into the container
COPY auto_committer.py auto_committer.py

# Expose the port the app runs on
EXPOSE 5009

# Run the application
CMD ["python", "auto_committer.py"]
```
## Troubleshooting
1. Ensure Docker and Docker Compose are installed and running.
2. Verify that the .env file is correctly configured.
3. Check the log file specified by LAB_LOG_FILE_PATH for any errors.

## Contributing
Feel free to fork the repository and submit pull requests. Contributions and feedback are welcome!


## License
    Copyright 2024 Dhirendra Singh

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
    
## Conclusion
The Auto-Committer project is a powerful tool designed to streamline the process of keeping your Git repositories up-to-date with minimal effort. By leveraging Docker and Python, this application automates the committing and pushing of changes, ensuring that your codebase is consistently synchronized with your remote repository.

Whether you're managing a large project or just need a reliable way to keep your repositories current, the Auto-Committer provides a flexible and easy-to-deploy solution. With its configurable intervals and robust logging, you can maintain a seamless workflow and focus on what truly matters—coding and innovation.

For more information and the complete source code, visit the GitHub Repository.

Feel free to explore, modify, and contribute to this project. Your feedback and improvements can help make it even better. For any questions or support, don’t hesitate to reach out through [LinkedIn](https://www.linkedin.com/in/hanisntsolo/) or email me at [ds.pratap1997@gmail.com](ds.pratap1997@gmail.com) if you have any questions or suggestions.

Happy coding! 🎉
