HOW-TO
===

1. [Kubernetes](https://quarkus.io/guides/deploying-to-kubernetes)
2. [Quarkus neo4j](https://quarkus.io/guides/neo4j)
3. If need to create credentials for registry
```
kubectl create secret generic regcred --from-file=.dockerconfigjson=$HOME/.docker/config.json --type=kubernetes.io/dockerconfigjson
```
4. If need to retrieve credentials
```
kubectl run quarkus-quickstart --image=muallin/quarkus-neo4j --port=8080 --image-pull-policy=IfNotPresent  --overrides='{ "apiVersion": "apps/v1", "spec": { "imagePullSecrets": [{"name": "regcred"}] } }'
```
5. To expose endpoint
```
kubectl expose deployment quarkus-quickstart --type=NodePort
```

## Check 

- Must build dockerfile.jvm
- Must create and endpoint and a service ( work for kubernetes 1.13)
```yaml
 kind: "Service"
  apiVersion: "v1"
  metadata:
    name: "neo4j-service"
  spec:
    ports:
      -
        name: "bolt"
        protocol: "TCP"
        port: 7687
        targetPort: 7687 
---
  kind: "Endpoints"
  apiVersion: "v1"
  metadata:
    name: "neo4j-service" 
  subsets: 
    - addresses:
        - ip: "192.168.1.36" #The IP Address of the external web server
      ports:
        - port: 7687 
          name: "bolt"
```
- Looks like the first time, default neo4j password it not properly set. Connect to browser and change it
- Test with
  - Retrieve
```http request
  curl  $(minikube service quarkus-quickstart --url)/fruits
  ```
  - Create
```http request
curl -v -X POST $(minikube service quarkus-quickstart --url)/fruits -H 'Content-Type: application/json; charset=utf-8' -d $'{
  "name": "Apple"}'
```

