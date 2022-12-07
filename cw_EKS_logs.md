# To install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname  -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# To install kubectl
curl -LO "https://dl.k8s.io/release/$(curl  -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# To create a cluster
eksctl create cluster --name=test-cluster-1 \
            --version 1.24 \
            --region=us-east-1 \
            --zones=us-east-1a,us-east-1b \
            --without-nodegroup

# To associate IAM OIDC Provider 
eksctl utils associate-iam-oidc-provider \
   --region us-east-1 \
   --cluster test-cluster-1 \
   --approve

# To create a keypair named kube-kp
aws ec2 create-key-pair --key-name kube-kp

# To create a private Node Group with key-pair (kube-kp) and instnace type 't2.micro'.
eksctl create nodegroup --cluster=test-cluster-1 \
             --region=us-east-1 \
             --name=test-cluster-1-ng-private-1 \
             --node-type=t3.medium \
             --nodes=1 \
             --nodes-min=1 \
             --nodes-max=1 \
             --node-volume-size=10 \
             --ssh-access \
             --ssh-public-key=kube-kp \
             --managed \
             --asg-access \
             --external-dns-access \
             --full-ecr-access \
             --appmesh-access \
             --alb-ingress-access

cat <<EOF > nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF

kubectl apply -f nginx-deployment.yaml

# Atttach Cloudwatchfullaccess to node roles

kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml

ClusterName=test-cluster-1
RegionName=us-east-1
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
kubectl create configmap fluent-bit-cluster-info \
--from-literal=cluster.name=${ClusterName} \
--from-literal=http.server=${FluentBitHttpServer} \
--from-literal=http.port=${FluentBitHttpPort} \
--from-literal=read.head=${FluentBitReadFromHead} \
--from-literal=read.tail=${FluentBitReadFromTail} \
--from-literal=logs.region=${RegionName} -n amazon-cloudwatch

kubectl delete -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml
kubectl apply -f https://raw.githubusercontent.com/ad1mohan/SAA_Preperation/main/cm.yaml

kubectl exec -it nginx-deployment-6595874d85-hbwbj -- /bin/sh

echo Hello > /proc/1/fd/1
while true; do echo -n "This is a test of while loop " > /proc/1/fd/1;date ; sleep 1; done
