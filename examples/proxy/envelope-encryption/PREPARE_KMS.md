# Preparing a Key Management Service (KMS)

As discussed [here](./README.md) you must provide a KMS solution in order to use the Envelope Encryption Filter.
For Tech Preview, only HashiCorp Vault is supported.

## Preparing HashiCorp Vault

You may either use an existing instance of HashiCorp Vault you have available in your organisation, or deploy a development instance
HashoCorp Vault to your OpenShift Cluster which is good enough to try out the Filter.

## Using an Existing Vault Instance

### Prereqistes

You must have an HashCorp Vault instance running on premise or you have a HashiCorp HCP Vault or Enterprise.  Free tier HCP Vault Secrets is
*not* adequate.

* You must have adminstrative access to the Vault instance or Vault Namespace.
* You must know the Address of the Vault Server, and if, Vault Enterprise, the name of the Vault Namespace.
* Vault CLI installed.

### Configure the Existing Instance

1. Set the `VAULT_ADDR` (and `VAULT_NAMESPACE` if Enterprise) to refer to the Vault Environment.
   ```sh
   export VAULT_ADDR=https://<vault server>:8200
   export VAULT_NAMESPACE=<namespace(s)>
2. Login to Vault as the Adminstrator.
   ```sh
   vault login
   ```
3. Enable the Transit Secrets Engine.
   ```sh
   vault secrets enable transit
   ```
   If the Transit engine was already enabled, an error will be printed but this can be ignored.
4. To conclude follow the steps below to [Create the Vault Policy and Filter Vault Token](#create-the-vault-policy-and-filter-vault-token)

## Deploying a standalone development instance of Vault

This option will give you an development instance of HashiCorp Vault with ephemeral storage.  This is sufficient to help you evaluate the
Filter, but is *not* suitable for production or controlled test environments.

It is assumed that you'll be deploying the HashiCorp Vault feature to the same OpenShift Cluster where you'll be evaluating AMQ Stream Proxy.

### Prerequsites

* Administrative access to the OpenShift Cluster being used to evaluate AMQ Stream Proxy
* Helm CLI
* OpenShift CLI (oc)

### Deploying HashiCorp for Development

1. Log in to the OpenShift Cluster as the adminstrator
   ```sh
   oc login https://<openshift cluster> --username <cluster admin>
   ```
2. Create a Vault Root Token. Keep a note of the token assigned.
   ```sh
   export VAULT_TOKEN=$(cat /dev/urandom | LC_ALL=C tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)
   ```
3. Install Vault using Helm
   ```sh
   helm repo add hashicorp https://helm.releases.hashicorp.com
   helm install vault hashicorp/vault --create-namespace --namespace=vault  --values vault/helm-dev-values.yaml --set server.dev.devRootToken=${VAULT_TOKEN} --wait
   ```
4. Assign the `VAULT_ADDR` environment variable to point at the new instance.
   ```
   export VAULT_ADDR=$(oc get route -n vault vault --template='https://{{.spec.host}}')
   ```
5. To conclude follow the steps below to [Create the Vault Policy and Filter Vault Token](#create-the-vault-policy-and-filter-vault-token)

## Create the Vault Policy and Filter Vault Token

1. Create a Vault Policy for the Envelope Encryption Filter:
   ```sh
   vault policy write amqstreams_proxy_encryption_filter_policy vault/amqstreams_proxy_encryption_filter_policy.hcl
2. Create a Vault Token which will be used by the Envelope Encryption Filter:
   ```bash
   ENVELOPE_ENCRYPTION_TOKEN=$(vault token create -display-name "amqstreams-proxy encryption filter" -policy=amqstreams_encryption_filter_policy -no-default-policy -orphan -field=token)
   ```
3. Prepare a secret containing the Envelope Encryption Vault Token.
   ```bash
   oc create secret generic filter-vault-token -n proxy --from-literal=filter-vault-token.txt=${ENVELOPE_ENCRYPTION_TOKEN} --dry-run=client -o yaml > proxy-filter-vault-token.yaml
   ```
   The secret file `proxy-filter-vault-token.yaml` will be applied to the OpenShift Cluster later.
    
