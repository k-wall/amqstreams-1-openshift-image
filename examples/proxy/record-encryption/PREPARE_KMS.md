# Preparing a Key Management Service (KMS)

In order to use the Record Encryption Filter, you must provide a [KMS solution](./README.md).

# Fortanix DSM (Development)

This example assumes Fortanix DSM SaaS.

### Prerequisites

You must have 

* Fortanix DSM account
* Know the Fortanix DSM endpoint e.g. https://api.uk.smartkey.io
* Fortanix DSM CLI installed and have logged in 
* GNU sed

### CLI

```
python3 -m venv ./venv
. ./venv/bin/activate
pip3 install sdkms-cli

export FORTANIX_API_ENDPOINT=https://api.uk.smartkey.io
sdkms-cli user-login  --username xxxx@yyyy.zzz
```

### Create a Fortanix Group for the Topic Keys


```
sdkms-cli  create-group --name topic-keks
```

you'll need the group id later.


### Create a Fortanix App for use by Record Encryption and retrieve the API key

```
sdkms-cli create-app --name kroxylicious --default-group topic-keks --groups topic-keks
sdkms-cli get-app-api-key --name kroxylicious > fortanix-dsm.apikey
```

### 

1. Create a secret containing the Fortanix Api Key
   ```sh
   oc create secret generic proxy-encryption-kms-secret -n kafka-proxy --from-file=fortanix-dsm-apikey.txt=fortanix-dsm.apikey --dry-run=client -o yaml > base/proxy/proxy-encryption-kms-secret.yaml
   ```

2. Update the proxy config to refer to your Fortanix DSM instance:
   ```sh
      sed -i "s|\(endpointUrl:\).*$|\1 ${FORTANIX_API_ENDPOINT}|" */proxy/proxy-config.yaml
   ```  

## Cleaning up

```sh
sdkms-cli list-keys
# then delete the keys by kid
sdkms-cli  delete-key --kid 0e83e230-964c-4358-921a-5fa8d4b6ef88

```

Finally, delete the .apikey files:
```sh
rm *.apikey
```

