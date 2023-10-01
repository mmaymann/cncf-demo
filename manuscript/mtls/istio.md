# Mutual TLS (mTLS) And Network Policies With Istio

TODO: Intro

## Setup

```bash
# TODO: kapp-controller

export GITOPS_APP=$(yq ".gitOps.app" settings.yaml)

cat $GITOPS_APP/istio.yaml

cp $GITOPS_APP/istio.yaml infra/.

git add . 

git commit -m "Istio"

git push

kubectl --namespace istio-system get pods

# Wait until the Pods are created and ready
```

# Do

```bash
cat istio/namespace-production.yaml

cp istio/namespace-production.yaml apps/.

git add . 

git commit -m "Istio"

git push

kubectl get namespace production --output yaml

# Wait until the `istio-injection` label was added.

kubectl --namespace production delete pods \
    --selector app.kubernetes.io/name=cncf-demo

kubectl --namespace production get pods

# Istio side-car is now added to the Pods
```

## Authorization (mTLS)

```bash
kubectl --namespace production apply --filename istio/mtls.yaml

kubectl --namespace production --tty --stdin exec sleep \
    --container sleep -- sh

apk add -U curl

curl -s http://httpbin:8080/headers \
    | grep X-Forwarded-Client-Cert

# Cert shows that the communication between the Pods is
#   encrypted (mTLS)

exit
```

## Authentication

```bash
cat istio/peer-authentication.yaml

kubectl --namespace production apply \
    --filename istio/peer-authentication.yaml

kubectl --namespace production --tty --stdin exec sleep \
    --container sleep -- sh

curl http://cncf-demo.production:8080 -w "%{http_code}\n"

# Both apps are "meshed", so the communication is allowed.

exit

kubectl --namespace default apply --filename istio/mtls.yaml

kubectl --namespace default --tty --stdin exec sleep \
    --container sleep -- sh

apk add -U curl

curl http://cncf-demo.production:8080 -w "%{http_code}\n"

# The communication is not allowed since the Pod in the
#   `default` Namespace is NOT "meshed" and therefore not
#   authenticated.

exit
```

## Policies

```bash
cat istio/authorization-policy-deny.yaml

kubectl --namespace production apply \
    --filename istio/authorization-policy-deny.yaml

kubectl --namespace production --tty --stdin exec sleep \
    --container sleep -- sh

curl http://cncf-demo.production:8080

# The access was denied

exit

cat istio/authorization-policy-allow.yaml

kubectl --namespace production apply \
    --filename istio/authorization-policy-allow.yaml

kubectl --namespace production --tty --stdin exec sleep -- sh

curl http://cncf-demo.production:8080

# The access was allowed
```

## Destroy

```bash
kubectl --namespace production delete --filename istio/mtls.yaml

kubectl --namespace production delete \
    --filename istio/peer-authentication.yaml

kubectl --namespace default delete --filename istio/mtls.yaml

kubectl --namespace production delete \
    --filename istio/authorization-policy-allow.yaml
```

## Continue The Adventure

* [Mutual TLS And Network Policies](../scanning/README.md)