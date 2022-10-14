# frontend-assignment
Docker backend for the BY front end assignment

# Create CiCD

Check the "pipline.yml" file in the directory ".github/workflows"<br>
It is pretty much self-explanatory and I used date to add tags automatically.<br>

# Run and test on Minikube

First I tried running Docker Compose on my local machine. <br>
I had som trouble running the Ruby app (no experience here). <br>
I have fixed the Dockerfile, changed some Postgres conf and was able to run the app using DockerCompose. <br>
You can check these changes in Dockerfile and docker-compose.yml files. <br>

I then buit a docker image and pushed it to my own docker hub account. <br>
https://hub.docker.com/repository/docker/mahshaban95/frontend-assignment_web <br>

I installed minikube on my local machine. <br>

I used Kompose to convert docker compose file into helm chart. <br>
<code>kompose convert -c -o my-app-kompose</code> <br>

I fixed the errors associated with this converstion and the app worked correctly. <br>

There is still an error (db has to be migrated inside the app container) but I got a temp solution for it. <br>
The temp fix is to get into the container shell and migrate the DB manually:<br>

<code>kubectl exec -it <pod> -c <container> bash</code><br>
<code>bin/rails db:migrate RAILS_ENV=development</code><br>

The app then worked correctly. <br>

# Add ELK

I could not complete this step because I did not have time to work on it. <br>

# Add Prometheus and Grafana

<code>helm repo add prometheus-community https://prometheus-community.github.io/helm-charts</code><br>
<code>helm repo update</code><br>
<code>helm install prometheus prometheus-community/kube-prometheus-stack</code><br>

## Access Prometheus UI

We first get the port which the prometheus container is listening to from the logs:\n<br>
<code>kubectl logs pod/prometheus-prometheus-kube-prometheus-prometheus-0 -c prometheus</code><br>

Then, forward this port as speciied below <br>
<code>kubectl port-forward pod/prometheus-prometheus-kube-prometheus-prometheus-0 9090</code><br>

## Access Grafana

We first get the port which the grafana pod container is listening to from the logs:\n<br>
<code>kubectl logs pod/prometheus-grafana-8fc6c5976-7cszj -c grafana</code><br>

Then, forward this port as speciied below <br>
<code>kubectl port-forward deployment/prometheus-grafana 3000</code><br>

Then access grafana UI 172.0.0.1:3000<br>

Username is admin as specified by the container logs.<br>
Password can be retrieved by the following command:<br>

<code>kubectl get secret prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo</code><br>


## Add Exporter to get logs from the postgres DB

We will use helm to install postgress exporter.<br>
First we need to get the values that need to be configured for the exporter.<br>
<code>helm show values prometheus-community/prometheus-postgres-exporter > postgres-values.yaml</code><br>
Then, ovveride the postgres-values.yaml file (you can find it in the repo).<br>

Install using the changed value:<br>
<code>helm install postgres-exporter prometheus-community/prometheus-postgres-exporter -f postgres-values.yam</code><br>

Open Grafana as above and see the new targets for post gres added.



