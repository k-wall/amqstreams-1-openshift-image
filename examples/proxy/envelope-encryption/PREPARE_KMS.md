# Preparing a Key Management Service (KMS)

As discussed [here](./README.md) you must provide a KMS solution in order to use the Envelope Encryption Filter.
For Tech Preview, only HashiCorp Vault is supported.

## Preparing HashiCorp Vault

You may either use an existing instance of HashiCorp Vault you have available in your organisation, or deploy a development instance
HashoCorp Vault to your OpenShift Cluster which is good enough to try out the Filter.

## Using an Existing Instance

### Prereqistes

You must have an HashCorp Vault instance running on premise or you have a HashiCorp HCP Vault or Enterprise.  Free tier HCP Vault Secrets is
*not* adequate.

* You must have adminstrative access to the Vault instance or Vault Namespace.
* You must know the Address of the Vault Server, and if, Vault Enterprise, the name of the Vault Namespace.
* Vault CLI installed.

### Configure the Existing Instance

1. Set the `VAULT_ADDR` (and `VAULT_NAMESPACE` if Enterprise) to refer to the Vault Environment.
   ```
   export VAULT_ADDR=https://<vault server>:8200
   export VAULT_NAMESPACE=<namespace(s)>
2. Login to Vault as the Adminstrator
   `vault login`
3. Enable the Transit Secrets Engine.
   `vault secrets enable transit`
4. Create a Vault Policy for the Envelope Encryption Filter:
   ```bash
   vault policy write amqstreams_proxy_encryption_filter_policy - << EOF
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
   ```
4. Create a Vault Token which will be used by the Envelope Encryption Filter:
   ```
   ENVELOPE_ENCRYPTION_TOKEN=$(vault token create -display-name "amqstreams-proxy encryption filter" -policy=amqstreams_encryption_filter_policy -no-default-policy -orphan -field=token)
   ```
   The token will be written to the ${ENVELOPE_ENCRYPTION_TOKEN} shell variable. You'll need the token when deploying the filter in the later step.

## Deploying a standalone development instance of Vault

This option will give you an development instance of HashiCorp Vault with ephemeral storage.  This is sufficient to help you evaluate the
Filter, but is *not* suitable for production or controlled test environments.

It is assumed that you'll be deploying the HashiCorp Vault feature to the same OpenShift Cluster where you'll be evaluating AMQ Stream Proxy.

### Prerequsites

* Administrative access to the OpenShift Cluster being used to evaluate AMQ Stream Proxy
* Helm CLI
* oc CLI

### Deploying HashiCorp for Development

1. Log into the OpenShift Cluster as the adminstrator
   `oc login https://<openshift cluster> --username <cluster admin>`
2. 
