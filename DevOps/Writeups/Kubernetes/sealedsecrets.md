# Integrating sealed secrets into CI/CD pipeline.

## Install via HELM on cluster
```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets -n kube-system sealed-secrets/sealed-secrets
```

## Install kubeseal CML on local device
```bash
# Arch (Out of date, dont use until unflagged)
sudo pacman -S kubeseal

# Manual for linux
KUBESEAL_VERSION='' # Set this to, for example, KUBESEAL_VERSION='0.23.0'
curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION:?}/kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz"
tar -xvzf kubeseal-${KUBESEAL_VERSION:?}-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal

# Dynamically for linux
# Fetch the latest sealed-secrets version using GitHub API
KUBESEAL_VERSION=$(curl -s https://api.github.com/repos/bitnami-labs/sealed-secrets/tags | jq -r '.[0].name' | cut -c 2-)

# Check if the version was fetched successfully
if [ -z "$KUBESEAL_VERSION" ]; then
    echo "Failed to fetch the latest KUBESEAL_VERSION"
    exit 1
fi

curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"

tar -xvzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal

sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

## Ensure you have kubectl setup on local machine

```bash
# Arch
sudo pacman -S kubectl

scp user@masternode:~/.kube/config ~/.kube/

kubectl get pods -A
```

## Create a secret locally
```bash
# Note the -n homelab for where you want the secret to go when unsealed
kubectl create secret generic pihole-secret -n homelab --from-literal=password=$(echo -n 'your-password-here' | base64) --dry-run=client -o yaml > pihole-secret.yaml

kubeseal -f pihole-secret.yaml -w sealed-pihole-secret.yaml --format yaml
# If you get error cannot get sealed secret service, it is because you changed the default name of the service. Just go on master node and check the service name. In my case it is sealed-secrets, so you can do the following,
kubeseal -f pihole-secret.yaml -w sealed-pihole-secret.yaml --controller-name sealed-secrets --format yaml

# Now you have a sealed json file that can be normally applied with kubectl apply -f mysealedsecret.yaml . This will then create a normal secret from the spec inside.
```