## Steps to deploy application to Google Kubernetes Engine
### Setup Kubernetes Engine Cluster
Use the command - gcloud container clusters create --machine-type n1-standard-2 --num-nodes 2 --zone us-central1-a --cluster-version latest adityadwkubernetescluste
### sa-logic
* Get the docker image from dockerhub - docker pull adityadw/sentiment-analysis-logic
* Tag the docker image - docker tag adityadw/sentiment-analysis-logic gcr.io/genuine-grid-327615/adityadw/sentiment-analysis-logic:1
* Push the docker image to google's container registery - docker push gcr.io/genuine-grid-327615/adityadw/sentiment-analysis-logic:1
* Configure the deployment.yaml file by giving name as sa-logic, replicas as 2, having a rolling update, surge pods as 1, and the container image poiniting to the one we pushed - adityadw/sentiment-analysis-logic and the containerPort to 5000 on which the python web app listens
* Launch the deployment using - kubectl apply -f sa-logic-deployment.yaml --record
* Configure the service for frontend by creating a yaml file of kind Service, app-selector as sa-logic, the port which container listens for as 5000 and the port via which one interacts with container as 80
* Deploy the service as - kubectl apply -f service-sa-logic.yaml
### sa-webapp
* Get the docker image from dockerhub - docker pull adityadw/sentiment-analysis-web-app
* Tag the docker image - docker tag adityadw/sentiment-analysis-web-app gcr.io/genuine-grid-327615/adityadw/sentiment-analysis-web-app:1
* Push the docker image to google's container registery - docker push gcr.io/genuine-grid-327615/adityadw/sentiment-analysis-web-app:1
* Configure the deployment yaml file similar to sa-logic but changing container image to adityadw/sentiment-analysis-web-app and container port on which the jar listens as 8080 and add a parameter SA_LOGIC_API_URL as http://sa-logic which will be used to interact with sa-logic service by the jar, this is achevied by using the kube-dns feature which resolves service name to corresponding service ip
* Launch the deployment using - kubectl apply -f sa-web-app-deployment.yaml --record
* Configure the service similar to above but adding spec as loadbalancer so that this can be exposed outside and map 8080 internal container port to 80 outside port
* Deploy the service using - kubectl apply -f service-sa-web-app-lb.yaml
### Getting IP  of sa-webapp for sa-frontend
* We need the IP for sa-frontend to reach sa-webapp, we get the IP from GUI under deployment details in Kubernetes engine for sa-webapp
* Change the App.js file for sa-frontend to reach this IP
* Run npm build to build the app
* Build the docker image again - docker build -f Dockerfile -t adityadw/sentiment-analysis-frontend:latest .
* Push the docker image to dockerhub - docker push adityadw/sentiment-analysis-frontend:latest
### sa-frontend
* Get the docker image from dockerhub - docker pull adityadw/sentiment-analysis-frontend
* Tag the docker image - docker tag adityadw/sentiment-analysis-frontend:latest gcr.io/genuine-grid-327615/adityadw/sentiment-analysis-frontend:1
* Push the docker image to google's container registery - docker push gcr.io/genuine-grid-327615/adityadw/sentiment-analysis-frontend:1
* Configure the deployment file as above but now setting container image as adityadw/sentiment-analysis-frontend:latest and container port to 80
* Launch the deploymet using - kubectl apply -f sa-frontend-deployment.yaml --record
* Configure the service as above mapping internal port 80 to external port 80
* Deploy the service using - kubectl apply -f service-sa-frontend-lb.yaml
## Screenshots
* kubernetes_web_app.PNG - screenshot showing webapp running on public IP
* kubernetes_front_end_service_details.PNG - front end servive details showing the public IP
* kubernetes_web_app_components.PNG - screenshot showing all the web app components deployed
## Docker-Hub Images URL
* sa-logic - https://hub.docker.com/r/adityadw/sentiment-analysis-logic
* sa-webapp - https://hub.docker.com/r/adityadw/sentiment-analysis-web-app
* sa-frontend - https://hub.docker.com/r/adityadw/sentiment-analysis-frontend
## Video Recording
Please see demo.mp4 file

## Steps to build and push docker images
* git clone https://github.com/rinormaloku/k8s-mastery.git
* In sa-frontend - npm run build
* docker build -f Dockerfile -t adityadw/sentiment-analysis-frontend .
* docker push adityadw/sentiment-analysis-frontend
* In sa-logic - docker build -f Dockerfile -t adityadw/sentiment-analysis-logic .
* docker push adityadw/sentiment-analysis-logic
* In sa-webapp - mvn install
* docker build -f Dockerfile -t adityadw/sentiment-analysis-web-app .
* docker push adityadw/sentiment-analysis-web-app

#### Note- for detailed report and screenshots please see report.pdf

## References
* https://github.com/rinormaloku/k8s-mastery (dockerfiles and deployment files are borrowed from here and then tweaked wherever required)
* https://www.freecodecamp.org/news/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882/

#
