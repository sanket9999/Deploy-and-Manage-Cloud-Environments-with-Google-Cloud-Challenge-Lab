# Deploy-and-Manage-Cloud-Environments-with-Google-Cloud-Challenge-Lab

Task 1:
(SSH)

cd /work/dm

sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network --config=prod-network.yaml


gcloud config set compute/zone us-east1-b

gcloud container clusters create kraken-prod \
          --num-nodes 2 \
          --network kraken-prod-vpc \
          --subnetwork kraken-prod-subnet

gcloud container clusters get-credentials kraken-prod

cd /work/k8s

for F in $(ls *.yaml); do kubectl create -f $F; done

(If Error occurs then -

open deployment-prod-backend.yaml & deployment-prod-frontend.yaml respectively and add,

apiVersion: apps/v1

spec:
  selector:
   matchLabels:
    app: sample

After that run,
for F in $(ls *.yaml); do kubectl create -f $F; done)

gcloud config set compute/zone us-east1-b

gcloud compute instances create kraken-admin --network-interface="subnet=kraken-mgmt-subnet" --network-interface="subnet=kraken-prod-subnet"




Task 3:
(Shell)

gcloud config set compute/zone us-east1-b

gcloud container clusters get-credentials spinnaker-tutorial

DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \     
    -o jsonpath="{.items[0].metadata.name}")


(If Error occurs then -

gcloud container clusters create spinnaker-tutorial \
    --machine-type=n1-standard-2

gcloud iam service-accounts create spinnaker-account \
    --display-name spinnaker-account

export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')
    
export PROJECT=$(gcloud info --format='value(config.project)')

gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin \
    --member serviceAccount:$SA_EMAIL

gcloud iam service-accounts keys create spinnaker-sa.json \
     --iam-account $SA_EMAIL

gcloud pubsub topics create projects/$PROJECT/topics/gcr

gcloud pubsub subscriptions create gcr-triggers \
    --topic projects/${PROJECT}/topics/gcr
    
export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')
    
gcloud beta pubsub subscriptions add-iam-policy-binding gcr-triggers \
    --role roles/pubsub.subscriber --member serviceAccount:$SA_EMAIL    

wget https://get.helm.sh/helm-v3.1.0-linux-amd64.tar.gz


tar zxfv helm-v3.1.0-linux-amd64.tar.gz

cp linux-amd64/helm .

kubectl create clusterrolebinding user-admin-binding \
    --clusterrole=cluster-admin --user=$(gcloud config get-value account)


kubectl create clusterrolebinding --clusterrole=cluster-admin \
    --serviceaccount=default:default spinnaker-admin

./helm repo add stable https://kubernetes-charts.storage.googleapis.com
./helm repo update

export PROJECT=$(gcloud info \
    --format='value(config.project)')

export BUCKET=$PROJECT-spinnaker-config

gsutil mb -c regional -l us-east1 gs://$BUCKET



export SA_JSON=$(cat spinnaker-sa.json)
export PROJECT=$(gcloud info --format='value(config.project)')
export BUCKET=$PROJECT-spinnaker-config
cat > spinnaker-config.yaml <<EOF
gcs:
  enabled: true
  bucket: $BUCKET
  project: $PROJECT
  jsonKey: '$SA_JSON'

dockerRegistries:
- name: gcr
  address: https://gcr.io
  username: _json_key
  password: '$SA_JSON'
  email: 1234@5678.com
# Disable minio as the default storage backend
minio:
  enabled: false
# Configure Spinnaker to enable GCP services
halyard:
  additionalScripts:
    create: true
    data:
      enable_gcs_artifacts.sh: |-
        \$HAL_COMMAND config artifact gcs account add gcs-$PROJECT --json-path /opt/gcs/key.json
        \$HAL_COMMAND config artifact gcs enable
      enable_pubsub_triggers.sh: |-
        \$HAL_COMMAND config pubsub google enable
        \$HAL_COMMAND config pubsub google subscription add gcr-triggers \
          --subscription-name gcr-triggers \
          --json-path /opt/gcs/key.json \
          --project $PROJECT \
          --message-format GCR
EOF



./helm install -n default cd stable/spinnaker -f spinnaker-config.yaml \
           --version 2.0.0-rc9 --timeout 10m0s --wait
           

export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
    -o jsonpath="{.items[0].metadata.name}")

kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &   )

open port 8080

gsutil -m cp -r gs://spls/gsp114/sample-app.tar .

mkdir sample-app
tar xvf sample-app.tar -C ./sample-app

cd sample-app

git config --global user.email "$(gcloud config get-value core/account)"

git config --global user.name "[USERNAME]"   //user id of qwiklab

git init

git add .

git commit -m "Initial commit"

gcloud source repos create sample-app

git config credential.helper gcloud.sh

export PROJECT=$(gcloud info --format='value(config.project)')

git remote add origin https://source.developers.google.com/p/$PROJECT/r/sample-app

git push origin master

///////////////////////////////////////
[Navigation Menu > Source Repositories.

View All Repositories

sample-app. ]

NOT NECESSARY
////////////////////////////////////////////////////

Navigation menu > Cloud Build > Triggers

Create trigger.


Name:sample-app-tags

Event: Push new tag

Select your newly created sample-app repository.

Tag: v.*

Build configuration:   Cloud Build configuration file (yaml or json)

Cloud Build configuration file location:   /cloudbuild.yaml

then create

///////////////////////////////////////////////////

export PROJECT=$(gcloud info --format='value(config.project)')

gsutil mb -l us-central1 gs://$PROJECT-kubernetes-manifests

gsutil versioning set on gs://$PROJECT-kubernetes-manifests

sed -i s/PROJECT/$PROJECT/g k8s/deployments/*

git commit -a -m "Set project ID"


git tag v1.0.0

git push --tags


curl -LO https://storage.googleapis.com/spinnaker-artifacts/spin/1.14.0/linux/amd64/spin

chmod +x spin


./spin application save --application-name sample \
                        --owner-email "$(gcloud config get-value core/account)" \
                        --cloud-providers kubernetes \
                        --gate-endpoint http://localhost:8080/gate
                        
export PROJECT=$(gcloud info --format='value(config.project)')
sed s/PROJECT/$PROJECT/g spinnaker/pipeline-deploy.json > pipeline.json
./spin pipeline save --gate-endpoint http://localhost:8080/gate -f pipeline.json

//////////////////////////////////////////////////


sed -i 's/orange/blue/g' cmd/gke-info/common-service.go

git commit -a -m "Change color to blue"

git tag v1.0.1

git push --tags




git revert v1.0.1

git tag v1.0.2

git push --tags







////If you still get error run this:


gcloud config set compute/zone us-east1-b

gcloud source repos clone sample-app

cd sample-app

touch a

git config --global user.email "$(gcloud config get-value account)"

git config --global user.name "Student"

git commit -a -m "change"

git tag v1.0.1

git push --tags
