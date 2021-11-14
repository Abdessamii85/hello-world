# Build "Hello World" + Jenkins Pipeline 

## Problem specification

```
Design and code a simple "Hello World" application that exposes the following HTTP-based APIs: 

Description: Saves/updates the given user's name and date of birth in the database. 
Request: PUT /hello/<username> { "dateOfBirth": "YYYY-MM-DD" }
Response: 204 No Content
 
Note:
<usemame> must contains only letters. 
YYYY-MM-DD must be a date before the today date. 

Description: Returns hello birthday message for the given user 
Request: Get /hello/<username> 
Response: 200 OK 

Response Examples: 
A. If username's birthday is in N days: { "message": "Hello, <username>! Your birthday is in N day(s)" } 
B. If username's birthday is today: { "message": "Hello, <username>! Happy birthday!" } 

Note: Use storage/database of your choice. The code should have at least one unit test. 
```



## Project structure 

- root directory contain  Jenkins Groovy pipeline file
- server: contains HTTP server sources
- server/tests: contains pytest's unit tests
- server/Dockerfile: Dockerfile of server

## How to run and test via docker
```bash
docker run -d -p 8081:8080 --name python-test abdessamii/user-py:${versionTag}
# exemple docker run -d -p 8081:8080 --name python-test  abdessamii/user-py:0.1.2
# see if the Hello world application works [http://localhost:8081/hello/Charlie](http://localhost:8081/hello/Charlie)

```

## How to run and test locally

- Run server ```python main.py``` 
- Run test without logging ```pytest -q``` 
- Run test without logging ```pytest -q -c pytest_logs.ini``` 

<img src="/home/abdessamii/Desktop/test.png">

You may test server manually, following link [http://localhost:8080/hello/Charlie](http://localhost:8080/hello/Charlie)

<img src="/home/abdessamii/Desktop/web.png">

## Prerequisites
```bash
pip install -r requirements.txt
```

## Setting up

### OSX
```bash
brew update
brew install python
pip install -U pytest
pytest --version
pip install -r requirements.txt
```

## Prerequisites for Pipeline CI with Jenkins
 slave Jenkins needs to have the following software istalled
 - [Docker](https://docs.docker.com/get-docker/) : for building the image and push to the dockerhub registry
 - [Trivy](https://github.com/aquasecurity/trivy#abstract) : Scan Image for Vulnerabilities
 - [junit](https://junit.org/junit4/) :  Unit testing frameworks.

## Continuous integration Pipeline
the Jenkinsfile is well commented, 
- This pipeline build automatically the abdessamii/user-py  image from a python:3.7.4 official image.
- Run unit test with junit against it
- scan the image for security issues, if critical or High Vulnerabilities the pipeline fail
- Push the image to dockerhub registry 
- send a email notification with the Build result