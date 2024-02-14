# AMQ Streams Proxy Envelope Encryption, exposed by load balancer

In this example, an instance of Apache Kafka is deployed using AMQ Stream.  The instance is proxied using
AMQ Streams Proxy configured with Envelope Encyption.  The proxy is exposed off cluster using a Kubernetes
Service.


# Prerequsistes

* [KMS is prepared](../PREPARE_KMS.md).
* Vault CLI

* Administrative access to the OpenShift Cluster being used to evaluate AMQ Stream Proxy
* OpenShift CLI (oc)
* AMQ Stream Operator is installed namespace wide
* Apache Kafka CLI

# Deploying the Example

1. Deploy the Example
   ```sh
   oc apply -k base
   ```
2. Apply the `envelope-encryption-vault-token-secret.yaml` created durin the KMS preparation step.
   ```sh
   oc apply -n proxy -f ../envelope-encryption-vault-token-secret.yaml
   ```
3. Get the external address of the proxy service
   ```sh
   LOAD_BALANCER_ADDRESS=$(oc get service -n proxy proxy-service --template='{{(index .status.loadBalancer.ingress 0).hostname}}')
   ```
4. Now update the `brokerAddressPattern:` to match the `LOAD_BALANCER_ADDRESS`.
   ```sh
     sed -i  "s/\(brokerAddressPattern:\).*$/\1 ${LOAD_BALANCER_ADDRESS}/" base/proxy/proxy-config.yaml
   ```
5. Reapply and bounce
   ```sh
      oc apply -k base && oc delete pod -n proxy
   ```

# Try out the example

1. Create a key for topic `trades` in the KMS:
   ```
   vault write -f transit/keys/KEK_trades
   ```
2. Create a topic `trades` on the cluster, via the proxy:
   ```
   kafka-topics.sh --bootstrap-server ${LOAD_BALANCER_ADDRESS}:9092 --create -topic trades
   ```
3. 
   ```
   echo 'IBM:100\nAPPLE:99' | kafka-console-producer --bootstrap-server ${LOAD_BALANCER_ADDRESS}:9092 -topic trades
   ```

4. 
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

