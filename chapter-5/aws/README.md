# AWS Cloud Provider

---
_🌍 Available in_: [English](README.md) | [中文 (Chinese)](README-zh.md)

> **Note:** Brought to you by the fantastic cloud-native community's [ 🌟 contributors](https://github.com/salaboy/platforms-on-k8s/graphs/contributors)!

---


In this step-by-step tutorial, we will use Crossplane to Provision Redis, PostgreSQL, and Kafka in AWS.

Then install the Crossplane AWS provider: 
```shell
kubectl apply -f provider/aws-provider.yaml
```

After a few seconds, if you check the configured providers, you should see the Helm `INSTALLED` and `HEALTHY`: 

```shell
> kubectl get providers.pkg.crossplane.io
NAME                             INSTALLED   HEALTHY   PACKAGE                               AGE
crossplane-provider-aws         True        True      crossplane/provider-aws:v0.21.2       49s
```

Now we are ready to install our Databases and Message Brokers Crossplane compositions to provision all the components our application needs to work.

## App Infrastructure on demand using Crossplane Compositions

We need to install our Crossplane Compositions for our Key-Value Database (Redis), our SQL Database (PostgreSQL) and our Message Broker(Kafka). 

```shell
kubectl apply -f resources/
```

The Crossplane Composition resource (`app-database-redis.yaml`) defines which cloud resources need to be created and how they must be configured together. The Crossplane Composite Resource Definition (XRD) (`app-database-resource.yaml`) defines a simplified interface that enables application development teams to quickly request new databases by creating resources of this type.

Check the [resources/](resources/) directory for the Compositions and the Composite Resource Definitions (XRDs). 

Create a text file containing the AWS account aws_access_key_id and aws_secret_access_key.

```text
[default]
aws_access_key_id = 
aws_secret_access_key = 
```

Create a Kubernetes secret with the AWS credentials. 

```shell
kubectl create secret \
generic aws-secret \
-n crossplane-system \
--from-file=creds=./aws-credentials.txt
```

Create a ProviderConfig 

```shell
kubectl apply -f aws-provider-config.yaml
```

### Let's provision Application Infrastructure

We can provision a new Key-Value Database for our team to use by executing the following commands to create all the infrastructure necessary: 

```shell
kubectl apply -f my-db-keyvalue.yaml
kubectl apply -f my-db-sql.yaml
kubectl apply -f aws-messagebroker-kafka.yaml
```

## Let's deploy our Conference Application

Ok, now that we have our two databases and our message broker running, we need to make sure that our application services connect to these instances. The first thing we need to do is disable the Helm dependencies defined in the Conference Application chart so that when the application gets installed, don't install the databases and the message broker. We can do this by setting the `install.infrastructure` flag to `false`.

For that, we will use the `app-values.yaml` file containing the configurations for the services to connect to our newly created databases:

```shell
helm install conference oci://registry-1.docker.io/salaboy/conference-app --version v1.0.0 -f app-values.yaml
```

Make sure to fill in the commented out aspects of the yaml file based off values from newly created AWS infrastructure.

The `app-values.yaml` content looks like this: 
```shell
install:
  infrastructure: false
frontend:
  kafka:
    url: #aws-kafka-endpoint
agenda:
  kafka:
    url: #aws-kafka-endpoint
  redis: 
    host: #aws-redis-endpoint
    secretName: #aws-redis-password
c4p: 
  kafka:
    url: #aws-kafka-endpoint
  postgresql:
    host: #aws-psql-endpoint
    secretName: #aws-psql-secret

notifications: 
  kafka:
    url: #aws-kafka-endpoint
```
