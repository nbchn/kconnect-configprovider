# Example for Kafka Connect with Vault Config Provider

1. Install Vault
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault --set='server.ha.enabled=true' --set='server.ha.raft.enabled=true' --set='injector.enabled=false'
```

2. Install Confluent Operator
```
helm repo add confluentinc https://packages.confluent.io/helm
helm upgrade --install operator confluentinc/confluent-for-kubernetes
```

3. Configure Vault
```
kubectl exec vault-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
VAULT_UNSEAL_KEY=$(cat cluster-keys.json | jq -r ".unseal_keys_b64[]")
kubectl exec vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY
CLUSTER_ROOT_TOKEN=$(cat cluster-keys.json | jq -r ".root_token")
```

4. Create a secret in Vault
```
kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh
> vault secrets enable -path=secret kv-v2
> vault kv put secret/devwebapp/config username='giraffe' password='salsa' topic='pageviews'
```

5. Deploy the Confluent platform
```
# Update token in platform.yml and vault address
kubectl apply -f platform.yml
```

6. Create the topic for the datagen connector
```
kubectl apply -f topic.yml
```

5. Deploy the datagen connector
```
kubectl apply -f datagen.yml
```
