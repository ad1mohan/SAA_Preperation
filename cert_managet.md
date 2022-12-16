# Create cluster with private worker nodes


# Install Open SSL
sudo yum install openssl -y

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh

# Install repo
helm repo add jetstack https://charts.jetstack.io

# Update repo
helm repo update

# Install cert manager
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.10.1 \
  --set installCRDs=true
