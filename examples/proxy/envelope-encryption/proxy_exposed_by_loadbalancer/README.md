
# Assumes

Vault CLI
oc command line
kafka CLIs

# Operators

AMQ Streams
Cert Manager

## Vault

If you do not already have an instance of Vault available, you can deploy an development server.  The instance is
single server and epheremal storage so key material will be lost on each restart. It is not suitable for production use!


```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
# TODO make user choose a root token
helm install vault hashicorp/vault --create-namespace --namespace=vault
```

Get the URL of the Vault Console and External API endpoint:
```bash
oc get routes -n vault vault --template='https://{{.spec.host}}'
```

Now configure Vault:

```bash
export VAULT_ADDR=$(oc get routes -n vault vault --template='https://{{.spec.host}}')
vault login
vault secrets enable transit

vault policy write kroxylicious_encryption_filter_policy - << EOF
path "transit/keys/KEK_*" {
capabilities = ["read"]
}
path "/transit/datakey/plaintext/KEK_*" {
capabilities = ["update"]
}
path "transit/decrypt/KEK_*" {
capabilities = [ "update"]
}
EOF

# TODO create the token in a secret so we can mount it straight in the kroxy deployment
export PROXY_VAULT_TOKEN=$(vault token create -display-name "kroxylicious encryption filter"  -policy=kroxylicious_encryption_filter_policy  -no-default-policy  -orphan -field=token)


```


```bash
oc apply -k overlays/selfsigned
```

# ROSA- why do I need to do this? 

oc annotate service -n proxy proxy-service "service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags=$(oc get service -n openshift-ingress  router-default --template='{{(index .metadata.annotations "service.beta.kubernet
es.io/aws-load-balancer-additional-resource-tags")}}')"

Now we can get the external address of the service

BROKER_ADDRESE=$(oc get service -n proxy proxy-service --template='{{(index .status.loadBalancer.ingress 0).hostname}}')

Now update the kroxy config to use the 

brokerAddressPattern: <whatever>

Now bounce kroxy...

Create a key

# Publish
# Consume
# Verify on cluster
kubectl -n kafka run consumer -ti --image=quay.io/kroxylicious/kaf --rm=true --restart=Never -- kaf consume foo -b my-cluster-kafka-bootstrap:9092



# Cleaning up

```bash
helm uninstall vault --namespace vault
oc apply -d overlays/selfsigned
```

