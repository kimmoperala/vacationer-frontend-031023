# Vacationer frontend
UI example 1, front page:
![Untitled](https://github.com/user-attachments/assets/247ace3d-b3ff-4b0c-a6d4-db75d37a64b3)

UI example 2, viewing own holidays:

![Untitled2](https://github.com/user-attachments/assets/c6ff72fa-f06f-4257-bdd7-0f63d2b86ba6)
UI example 3 and 4, popup window for adding holiday and its dates:

![Screenshot 2025-06-13 at 16 55 57](https://github.com/user-attachments/assets/6626a3dc-7104-4595-80d3-1512dc7565e2)
![Screenshot 2025-06-13 at 16 55 18](https://github.com/user-attachments/assets/03cc30c8-6c5a-4665-953a-d471cbab6100)

## Overview
- Vacationer is work holiday pre-planning tool. The application serves as a planning, monitoring and coordinating tool for staff holidays.

- Monthly calendar view of holidays, includes filtering of all users / only users with holidays.

- CRUD operations for vacations, users, teams. Recoverable deletion for users and teams.

- Workers can be added to customer or project teams which can be used as a filter in holiday calendar.

- Anyone can create teams. Only team members can edit and delete team

- Sends weekly Slack notifications of workers who are on vacation on the following weeks. Example messages:

  - Slack message example 1, public holidays and one user on holiday:<br><img width="337" alt="Screenshot 2025-06-13 at 16 48 35" src="https://github.com/user-attachments/assets/bd790a83-9eae-42f4-80d5-6b2c6db655d4" />

  - Slack message example 2, multiple users on holiday.<br><img width="1200" alt="Screenshot 2025-06-13 at 16 49 16" src="https://github.com/user-attachments/assets/dd4b28d3-9947-4b53-82fb-e2fd9fedea8e" />

- Calendar settings for user (colors, holiday symbols with “short” emojis accepted).

- Admin features:  add users, edit admin rights for users, delete users and teams from database, send Slack test messages, edit all teams

## Environment setup
1. Clone the repo.
2. Add code formatter Prettier to your IDE: https://prettier.io/docs/en/editors.html
3. Copy /.env.example file to /.env 
4. Run
```
npm install
```

### Run locally:
1. Start the frontend to port 3000 with
```
npm start
```

# Original documentation:

## Architecture
MERN-stack (MongoDB, Express.js, React, Node.js)

AWS (EKS + ECR + DocumentDB), Kubernetes

### Admins 
To get admin rights for QA or PROD, contact current Admins.

## Overview
- Vacationer is work holiday pre-planning tool for teams. The application serves as a planning, monitoring and coordinating tool for staff holidays.
- Monthly calendar view of holidays, includes filtering of all users / only users with holidays.
- CRUD operations for vacations, users, teams. Recoverable deletion for users and teams.
- Workers can be added to customer or project teams which can be used as a filter in holiday calendar.
- Anyone can create teams. Only team members can edit and delete team
- Sends weekly Slack notifications of workers who are on vacation on the following weeks.
- Calendar settings for user (colors, holiday symbols with “short” emojis accepted).
- Admin features:  add users, edit admin rights for users, delete users and teams from database, send Slack test messages, edit all teams

## Prerequisites
- Company VPN required.
- To log in, you need work Github account.
- Minimum requirement for the login is a membership of a Github organization

## Endpoints
- Production version: <PROD url hidden>
- QA version: <QA url hidden>/

## API
To access Swagger API documentation you need to be logged in to the application first.
- https://<PROD url hidden>/api/api-docs/
- http://<QA url hidden>/api/api-docs/
- localhost:3001/api/api-docs/ (local)

You need to be authorized (logged in) to make API calls. After authorized, you can test API with Swagger or with Postman.

## Version number
Both frontend and backend have version numbers: frontend has the version in footer, backend has the version in API documentation, as the version in Swagger (see picture)
Logic of versions:
- PROD has the version set in the git tag during deployment.
- QA has SHA-number of last commit.

## Third party components
- Calendar view: TanStack Table https://github.com/tanstack
- Date picker: React Date Picker https://github.com/Hacker0x01/react-datepicker
- Settings color picker: React Color https://github.com/casesandberg/react-color
- Nager.Date API at https://date.nager.at/
  - Provides public holidays of Finland to Calendar. Public holidays are saved in a cache variable in backend as an array of {year: public holidays} objects. If public holidays are not up to date in Vacationer, you need to empty the cache variable. Then new values will be retrieved by the backend from the Nager API.

## Dependabot
- Dependabot is set on both frontend and backend repos
- Confirm PRs created by Dependabot. Merge and test on QA before taking to PROD.

## AWS
### DocumentDB
DocumentDB is used to store data in both PROD and QA.

There are two database users. Their passwords can be found from parameter store.
- QA: qa-db-user
- PROD: prod-db-user

### QA and PROD database

```
ssh ubuntu@<Vacationer-PROD-db-connector IP address>
mongosh --tls --host <url hidden> --tlsCAFile rds-combined-ca-bundle.pem --username <qa-db-user or prod-db-user> --password
<password of qa-db-user or prod-db-user>
show dbs
use <database name>
```

#### Restoring databases
Databases have a setup of automatic 7 days snapshots.

## ECR
In deployment pipelines images are deployed to AWS ECR (Elastic Container Registry) repositories. ArgoCD uses the images in ECR repos.

ECR repos have Lifecycle policy of keeping 20 newest images (on QA 5 newest images)

### Checking the PROD version
Go to the PROD repository and check which image has the latest tag. The other tag on that image is the PROD version.

# ArgoCD
“Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.”
Since QA was built from PROD, environments are quite similar in namings. QA and PROD have their own namespaces.

# Deployments
Duration in April 2023: QA front 3 mins, back 2 mins. PROD front 10 mins, back 3 mins.

## QA
Qa pipeline runs automatically when a PR commit has been merged to “main” branch. Monitor that deployment(s) succeeded: <url hidden>

## PROD
Production has separate pipelines in argo-workflows which run CI when (PR) commit is pushed to “main” and CD when a new tag is pushed to “main”.

PROD pipelines build docker images and push them to ECR repo when a git tag is pushed to main branch. Built images are tagged with both the git tag and ‘latest’ tags. The ‘latest’ image tag is currently used and automatic deployment is done by argo workflows by deleting the old pods.

The Kubernetes manifest files are stored in cs-infra repo. ArgoCD manages the application. More documentation on how ArgoCD deployments and argo-workflows work is in the cs-infra repo.

### How-to deploy
1. Once changes have been tested on QA, tag the next version number to repository. Check the latest version number from BOTH repositories. Mark same tags for both frontend and backend if they both have changes.

```
git pull
git tag <version number>
git push --tags
```

2. Deployment should start automatically. Monitor that deployment(s) succeeded: <url hidden>

3. Create release notes in Github repos → tags → select tag → Create release from tag → Generate release notes → Publish release

4. Inform on Slack channel the new version number and latest changes

5. Test changes on PROD. If the tests fail, you can do a roll back 

### Deployment problems?
Check Github repository of your deployment and go to Settings > Webhooks

Then select Recent Deliveries where you will see a list of events. Check the event that was suppose to trigger the workflow. Then compare with the cs-infra repository code to see whether the sensor was suppose to catch the event.

### Roll back to older version
Currently the production build pipeline tags the image with both 'latest' and tag version 'x.y.z'.
The version used by the deployment is currently 'latest' and the new 'latest' image is taken into use by deleting the current pod (done by pipeline).
There are at least two ways to roll-back to an older version:

1. Run the deploy pipeline targeting the wanted version tag or commit. This will (re)build the image and move the 'latest' tag to that image.
a. go to <url hidden> and select deploy-vacationer workflow you want to edit and press submit
b. in Parameters revision, write “refs/tags/<TAG NAME>“ or “refs/commits/<COMMIT SHA>” and press Submit
c. deployment will start and you can monitor it

2. Edit the deployment manifest manually from argocd with the wanted image tag. Doing this will automatically restart the pod with the new image. 

a. go to <url hidden>
b. select the backend/frontend pod
c. go to live manifest, and edit “image” parameter’s ending to the wanted tag (should be “latest” by default)
d. deployment will start and you can monitor it
e. after deployment, edit the “image” parameter’s ending in live manifest back to “latest”

# Logs
Logs can be found from ArgoCD.  

Select the application (vacationer-qa / vacationer), then pod (frontend or backend) and then LOGS tab.

# Secrets and environment variables
## Secrets (Parameterstore)
Secrets are stored in ParameterStore.

There’s an AWS app user vacationer-production-user in Production account whose credentials are saved as AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.

DOCDB_CREDENTIALS are the credentials of the AWS DocumentDB.
SLACK_URI is the Slack webhook address (Slack channel). REACT_APP_ADDRESS is the backend address of environment.
GHUB_ID, GHUB_SECRET are used for Github authentication and and JWT_SECRET for JWT verification.

If you update secrets, in ArgoCD remove “vacationer-secret” node first (will recreate secrets) and then remove the pod node (vacationer-backend or vacationer frontend) where secrets are used (will recreate pod).

Kubernetes uses external-secrets to fetch secrets from ParameterStore and create Kubernetes secret resources. Secrets stored include:
Github access/deploy keys Github Oauth id/secret, ecr id/secret, Documentdb url, Slack url, jwt secret.

## Environment variables
There are also some other environment variables than secrets. They are set in pipelines.

# Slack messages
Vacationer can send messages to Slack channels: PROD to <channel name hidden> and QA & local to <other channel name hidden> . There is a weekly Slack message in PROD and a possibility to send test messages in QA and local (from Admin page).

Added collaborators (admins) can edit the vacationer-app Slack app in <url hidden>. In “Incoming Webhooks” you can edit the Slack channels where messages will be sent. If you make changes to the webhooks (Slack channels), remember to also edit the webhook urls in SLACK_URI secrets in QA or PROD respectively.


