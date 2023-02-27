# Week 1 — App Containerization

## Homework

### Containerize Backend

Initially, I verified that Docker extension is installed on Gitpod and then I created Dockerfile in backend-flask:
```dockerfile
WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
To verify that everything is working as expected before containarizing the backend of our application, I installed Python libraries, set environmental variables for frontend and backend URLs and run Python:

```sh
cd backend-flask
pip3 install -r requirements.txt
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
```

Following that, I unlocked port 4567, opened the link in browser and appended /api/activities/home to the url:

![Proof of working backend](/_docs/assets/backend_v1.png)

Indeed, the set up was done correctly, so I removed the newly created environmental variables.

After that, I built the backend container:
```
docker build -t  backend-flask ./backend-flask
```

### Containerize Frontend



### Multiple Containers



### Create Notifications Feature



### Run DynamoDB Local Container



### Run Postgres Container



## Challenges

### Run the dockerfile CMD as an external script

### Push and tag a image to DockerHub

### Use multi-stage building for a Dockerfile build

### Implement a healthcheck in the V3 Docker compose file

### Research best practices of Dockerfiles and attempt to implement it in your Dockerfile

### Learn how to install Docker on your localmachine and get the same containers running outside of Gitpod / Codespaces

### Launch an EC2 instance that has docker installed, and pull a container to demonstrate you can run your own docker processes.
