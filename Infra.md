## Step 1: Create Cluster

```sh
eksctl create cluster --name $clustername --version 1.32 --region ap-south-1 --nodegroup-name $node-group --node-type t3a.medium --nodes-min 1 --nodes-max 2 --node-volume-size 40 --asg-access --external-dns-access --full-ecr-access --alb-ingress-access --node-private-networking
```

## Step 2: Database Setup

- Set up: Standard
- PSQL
- Version: 16.4
- Choose the same VPC as the cluster.
- Security Group: Create a new one named `rds-sg` and allow all traffic from `0.0.0.0/0` for port `5432`.

### Get the Password, Username, and Host

## Step 3: Create a Container for PSQL Access

```sh
kubectl run -it --image=postgres:15.4 --restart=Never postgres-client -- bash
```

### If you want to connect again:

```sh
kubectl exec -it postgres-client -- bash
```

## Step 4: Log into Database from Client

```sh
psql -h $endpoint -p 5432 -d $database -U $username -W
```

## Step 5: Create a User for Database

```sql
CREATE USER  $username WITH PASSWORD '$password';
ALTER USER $username NOCREATEDB;
GRANT pg_read_all_data TO $username;
```

## Step 6: Create a PersistentVolumeClaim (Refer to @EFS installation document)

## Step 7: Create a Load Balancer (Refer to @nginx-ingress-controller document)

## Step 8: Deploy the Resources

### Navigate to `@service-BDS` folder

```sh
kubectl apply -f db_service.yaml  # Deploys DB endpoint as external service
kubectl apply -f app-BDS-service.yaml  # Deploys BDS app service in the cluster
kubectl apply -f db-creds.yaml  # Creates DB credentials
```

## Step 9: Deploy StatefulSet Resources

### Navigate to `@statefulset-BDS` folder

```sh
kubectl apply -f configmap-bds.yaml  # Deploys configmap for all pods
kubectl apply -f app-bds.yaml  # Deploys Bookdsapp pod
kubectl apply -f cron-bds.yaml  # Deploys Bookds-cron pod
kubectl apply -f statefulodoo1-bash.yaml  # Deploys bookds-bash pod
kubectl apply -f hpa-app-bds.yaml  # Deploys app pod for scaling
kubectl apply -f hpa-cron-bds.yaml  # Deploys cron pod for scaling
```

## Step 10: Check All Pods
```sh
kubectl get pods -all
```


## Step 11: Create Database

### Get into the `store-bash` or `portal-bash` container

```sh
kubectl exec -it $container_name bash
```

### Create Database from Command Line

```sh
odoo --db_host=db --db_user=$username --db_password=$db_password --db_port=5432 -d $databasename--without-demo=all
```

### Exit the Container and Log into PostgreSQL RDS

```sh
kubectl exec -it postgres-client -- bash
```

### Remove Database Creation Access

```sql
ALTER USER $username NOCREATEDB;
```

### Exit the Pod

Type `exit` or press `Ctrl+D`

### Restart the Pod

```sh
kubectl rollout restart statefulset
```

---


```

