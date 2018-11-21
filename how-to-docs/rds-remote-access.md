# RDS Remote Access
Currently RDS instances created by the acp team can only be accessed from within the cluster. By following this guide you can setup a proxy within your namespace which will allow you to access your postgres instance locally.

## Step 1

Run a postgres client container to proxy / connect through. Below is an example deployment to create this, which refers to the RDS credentials secret in your namespace:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: postgres-client
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: postgres-client
    spec:
      containers:
      - name: postgres-client
        image: quay.io/ukhomeofficedigital/postgres-client:v0.1
        resources:
          limits:
            cpu: 400m
            memory: 400Mi
          requests:
            cpu: 50m
            memory: 200Mi
        securityContext:
          runAsNonRoot: true
        command:
          - /usr/bin/nc
          - -v
          - -lk
          - -p
          - "5432"
          - -e
          - /usr/bin/nc
          - ${DB_HOST}
          - ${DB_PORT}
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              key: username
              name: <rds-secret>
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: <rds-secret>
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              key: endpoint
              name: <rds-secret>
        - name: DB_PORT
          valueFrom:
            secretKeyRef:
              key: port
              name: <rds-secret>
        resources:
          limits:
            cpu: 250m
            memory: 256Mi
```

## Step 2
Run port-forwarding via the kubectl client tool (the user would need to get the id of the running postgres-client container that is created via the above deployment)

```
# list the running pods in the namespace
kubectl --namespace <namepace> get pods

# Run port-forward connecting to the postgres-client pod that you see from the above command
kubectl --namespace <namespace> port-forward <postgres-container-id> 5432
```

## Step 3
Connect using psql or pgadmin to localhost:

```
# The database name is available in your kubernetes secret
psql -h localhost -p 5432 -U root <database-name>
```
