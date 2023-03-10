# Week 1 — App Containerization

## Homework

### Containerize Backend

1. Verified that Docker extension is installed on Gitpod and created Dockerfile in backend-flask:
```dockerfile
WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
2. To verify that everything is working as expected before containarizing the backend of our application, installed Python libraries, set environmental variables for frontend and backend URLs and run Python:

```sh
cd backend-flask
pip3 install -r requirements.txt
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
```

3. Unlocked port 4567, opened the link in browser, appended /api/activities/home to the url and confirned that the set up was done correctly:

![Proof of working backend](/_docs/assets/backend_v1.png)

4. Built the backend container and run it:
```sh
docker build -t  backend-flask ./backend-flask
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```
5. Verified that the set up was done correctly:

![Proof of working backend v2](/_docs/assets/backend_v2.png)

### Containerize Frontend

1. Ran NPM Install to copy the contents of node_modules:
```sh
cd frontend-react-js
npm i
```
2. Created a new Dockerfile in frontend-react-js folder:
```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```
3. Built the frontend container with docker-compose.yml (described in the next section), otherwise the commands for the container build and run would have been exactly the same as for the backend (but with port 3000).

### Multiple Containers

1. Created docker-compose.yml at the root of my project:
```yml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```
2. Built the containers:
```sh
docker compose up
```
3. Unlocked port 3000, opened the link in browser and verified that everything is working as expected:

![Proof of working app](/_docs/assets/docker_compose.png)

### Create Notifications Feature

#### Backend

1. Created a new Cruddur account and confirmed my email with default (1234) password:

![Proof of sign up](/_docs/assets/account.png)

2. Installed OpenAPI extension, opened openapi file in backend-flask folder and added a new path to the existing code:
```yaml
  /api/activities/notifications:
    get:
      description: 'Return a feed of activity for all of those that I follow'
      tags:
        - activities
      parameters: []
      responses:
        '200':
          description: Returns an array of activities
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Activity'
```
3. Did two modifications to app.py file following the home_activities example:
```py
from services.notifications_activities import *
```
```py
@app.route("/api/activities/notifications", methods=['GET'])
def data_notifications():
  data = NotificationsActivities.run()
  return data, 200
```
4. Created notifications_activities.py in backend-flask/services folder following the home_activities.py example:
```py
from datetime import datetime, timedelta, timezone
class NotificationsActivities:
  def run():
    now = datetime.now(timezone.utc).astimezone()
    results = [{
      'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
      'handle':  'Daria Korobenko',
      'message': 'I am the best!',
      'created_at': (now - timedelta(days=2)).isoformat(),
      'expires_at': (now + timedelta(days=5)).isoformat(),
      'likes_count': 5,
      'replies_count': 1,
      'reposts_count': 0,
      'replies': [{
        'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
        'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  'Coco',
        'message': 'Yes, you are!',
        'likes_count': 0,
        'replies_count': 0,
        'reposts_count': 0,
        'created_at': (now - timedelta(days=2)).isoformat()
      }],
    }
    ]
    return results
```
5. Unlocked port 4567, opened the link in browser, appended /api/activities/notifications to the url and verified that everything is working as expected:

![Proof of notifications backend](/_docs/assets/backend_notifications.png)

#### Frontend

1. Added two modifications to App.js file in frontend-react-js/src folder following the example of HomeFeedPage:
```js
import NotificationsFeedPage from './pages/NotificationsFeedPage';
```
```js
{
    path: "/notifications",
    element: <NotificationsFeedPage />
  },
```
2. Created NotificationsFeedPage.js and NotificationsFeedPage.css in frontend-react-js/src/pages folder.

3. Copied the code from HomeFeedPage.js to NotificationsFeedPage.js and added some adjustments (commented the modified rows):
```js
import './NotificationsFeedPage.css';  //here
import React from "react";

import DesktopNavigation  from '../components/DesktopNavigation';
import DesktopSidebar     from '../components/DesktopSidebar';
import ActivityFeed from '../components/ActivityFeed';
import ActivityForm from '../components/ActivityForm';
import ReplyForm from '../components/ReplyForm';

// [TODO] Authenication
import Cookies from 'js-cookie'

export default function NotificationsFeedPage() {  //here
  const [activities, setActivities] = React.useState([]);
  const [popped, setPopped] = React.useState(false);
  const [poppedReply, setPoppedReply] = React.useState(false);
  const [replyActivity, setReplyActivity] = React.useState({});
  const [user, setUser] = React.useState(null);
  const dataFetchedRef = React.useRef(false);

  const loadData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/notifications`  //here
      const res = await fetch(backend_url, {
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setActivities(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };

  const checkAuth = async () => {
    console.log('checkAuth')
    // [TODO] Authenication
    if (Cookies.get('user.logged_in')) {
      setUser({
        display_name: Cookies.get('user.name'),
        handle: Cookies.get('user.username')
      })
    }
  };

  React.useEffect(()=>{
    //prevents double call
    if (dataFetchedRef.current) return;
    dataFetchedRef.current = true;

    loadData();
    checkAuth();
  }, [])

  return (
    <article>
      <DesktopNavigation user={user} active={'notifications'} setPopped={setPopped} />  //here
      <div className='content'>
        <ActivityForm  
          popped={popped}
          setPopped={setPopped} 
          setActivities={setActivities} 
        />
        <ReplyForm 
          activity={replyActivity} 
          popped={poppedReply} 
          setPopped={setPoppedReply} 
          setActivities={setActivities} 
          activities={activities} 
        />
        <ActivityFeed 
          title="Notifications"  //here
          setReplyActivity={setReplyActivity} 
          setPopped={setPoppedReply} 
          activities={activities} 
        />
      </div>
      <DesktopSidebar user={user} />
    </article>
  );
}
```
4. Unlocked port 3000, opened the link in browser and verified that everything is working as expected:

![Proof of notifications page](/_docs/assets/frontend_notifications.png)

### Add DynamoDB Local and Postgres

Modified docker-compose.yml to include databases:
```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js
  dynamodb-local:  # add dynamodb-local
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
  db:  # add postgres
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
volumes: # store the postgres data locally
  db:
    driver: local
```

#### Postgres

1. Installed postgres client into .gitpod.yml. Installed the PostgreSQL extension, clicked on Settings and added it to .gitpod.yml.

```yaml
tasks:
  ...
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
vscode:
  extensions:
    - 42Crunch.vscode-openapi
    - cweijan.vscode-postgresql-client2

```

2. Connected to Database server with
- username: postgres
- password: password

![Connection to DB server](/_docs/assets/database_server.png)

3. Verified that it is possible to use interactive terminal to work with the PostgreSQL database:

![psql](/_docs/assets/psql.png)

#### DynamoDB Local

1. Verified that it is working as expected by creating a new table called *Music*:
```sh
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```
2. Created a new item:
```sh
aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
    --return-consumed-capacity TOTAL  
```
3. Got the existing records from the table:

![Proof of working Dynamo DB](/_docs/assets/dynamodb.png)

## Challenges

### Run the dockerfile CMD as an external script

1. Created setup.sh in backend-flask folder:
```sh
#!/bin/bash
python3 -m flask run --host=0.0.0.0 --port=4567
```
2. Modidied Dockerfile in backend-flask folder:
```dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt 
RUN pip3 install -r requirements.txt

COPY setup.sh /
#change access permission
RUN chmod +x /setup.sh

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
#use script
CMD [ "/setup.sh"]
```
3. Built and run the containers:

![Proof of using script](/_docs/assets/built_script.png)

3. Verified the results:

![Verified using script](/_docs/assets/verified_script.png)

### Push and tag an image to DockerHub

1. Created a new account in DockerHub.
2. Checked the existing docker images and logged into my Docker account. Tagged *aws-bootcamp-cruddur-2023-backend-flask* image (version1.0) and pushed it to DockerHub.
```sh
docker images
docker login
docker tag aws-bootcamp-cruddur-2023-backend-flask kdo1404/cruddur-backend:version1.0
docker push kdo1404/cruddur-backend:version1.0
```
3. Opened DockerHub to verify the results:

![Proof of image in DockerHub](/_docs/assets/dockerhub_image.png)

### Use multi-stage building for a Dockerfile build

1. Modified Dockerfile in backend-flask folder to use multi-stage build by following [Docker documentation](https://docs.docker.com/build/building/multi-stage/).
Named a first stage as *builder* and used it as a new stage *build1*:
```dockerfile
# first stage - builder
FROM python:3.10-slim-buster AS builder

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

# second stage - build1
FROM builder AS build1

COPY setup.sh /
RUN chmod +x /setup.sh

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "/setup.sh"]
```
2. Ran *compose up* command to verify that everything is working as expected:

![Proof of multistage build](/_docs/assets/multistage_build.png)

### Implement a healthcheck in the V3 Docker compose file

1. *curl* is not installed by default in Alpine images, thus used *wget* command to implement a healthcheck to frontend-react-js in Docker compose file:

```yaml
healthcheck:
  test: wget --no-verbose --tries=1 --spider http://localhost:3000 || exit 1
  interval: 60s
  retries: 5
  start_period: 10s
  timeout: 10s
```

2. Verified that the image is indeed healthy:

![Proof of working healthcheck](/_docs/assets/healthcheck.png)

### Install Docker on your localmachine and get the same containers running outside of Gitpod

1. Cloned git repo to my laptop:

![Clone repo](/_docs/assets/clone_repo.png)

2. Checked *.gitpod.yml* and installed [Docker](https://docs.docker.com/desktop/install/windows-install/), [Node.js](https://nodejs.org/en/download/) and [PostgreSQL](https://www.postgresql.org/download/windows/) on my laptop (Windows 10 OS).

3. Opened VSCode and added Docker, OpenAPI and PostgreSQL extensions.

4. Installed NPM:
```
cd frontend-react-js
npm init
npm install
```
5. Changed frontent and backend URLs in Docker compose file to http://localhost:3000 and http://localhost:4567, accordingly.

6. Ran docker compose up and verified that everything is working as expected:

![Proof of working frontend](/_docs/assets/frontend_local.png)

![Proof of working backend](/_docs/assets/backend_local.png)

![Proof of running Docker](/_docs/assets/docker_local.png)

### Launch an EC2 instance that has docker installed and pull a container

1. Created a new instance via AWS EC2 Console and in *Advanced details - User data* section added Docker install script (following [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html)):

```sh
#!/bin/bash
sudo amazon-linux-extras install docker
sudo service docker start
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user
```

2. Connected to it with the help of .pem file and verified that Docker is installed:

```sh
ssh -i .\EC2_Tutorial.pem ec2-user@PUBLIC_IP
docker info
```

![Proof of working EC2](/_docs/assets/ec2.png)

3. Pulled an image of Cruddur backend from DockerHub. This image was pushed by me earlier for one of the challenges.

```sh
docker pull kdo1404/cruddur-backend:version1.0
```

![Proof of working EC2](/_docs/assets/ec2_docker.png)
