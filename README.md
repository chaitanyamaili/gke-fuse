# GKE with Fuse static

## Create cluster
```bash
gcloud container clusters create demo_gke_cluster \
    --addons GcsFuseCsiDriver \
    --cluster-version=1.25 \
    --region=us-central1 \
    --workload-pool=project_fuse.svc.id.goog
# get credentials
gcloud container clusters get-credentials demo-fuse-cluster --region us-central1 --project project_fuse
```

## Updating existing cluster

```bash
gcloud container clusters demo-fuse-cluster --update-addons GcsFuseCsiDriver=ENABLED --region us-central1 --project project_fuse
```

```bash
# create namespace
kubectl create namespace fuse-ns
kubectl create serviceaccount fuse-ksa --namespace fuse-ns

gcloud iam service-accounts create fuse-sa --project=project_fuse
gcloud storage buckets create gs://demo-bucket-fuse

gcloud storage buckets add-iam-policy-binding gs://demo-bucket-fuse \
    --member "serviceAccount:fuse-sa@project_fuse.iam.gserviceaccount.com" \
    --role "roles/storage.objectViewer"
gcloud storage buckets add-iam-policy-binding gs://demo-bucket-fuse \
    --member "serviceAccount:fuse-sa@project_fuse.iam.gserviceaccount.com" \
    --role "roles/storage.objectAdmin"

gcloud iam service-accounts add-iam-policy-binding fuse-sa@project_fuse.iam.gserviceaccount.com \
    --role roles/iam.workloadIdentityUser \
    --member "serviceAccount:project_fuse.svc.id.goog[fuse-ns/fuse-ksa]"

kubectl annotate serviceaccount fuse-ksa \
    --namespace fuse-ns \
    iam.gke.io/gcp-service-account=fuse-sa@project_fuse.iam.gserviceaccount.com
```

## Apply changes

```bash
kubectl apply -f static-provisiong.yaml -n fuse-ns
```