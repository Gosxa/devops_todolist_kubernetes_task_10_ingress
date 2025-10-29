1) Setup (kind + ingress)
# Create kind cluster with host ports mapped to 80/443
cat > cluster.yml <<'YML'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
YAML

kind create cluster --name todoapp --config cluster.yml

# Install NGINX Ingress (kind provider)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.3/deploy/static/provider/kind/deploy.yaml
kubectl wait -n ingress-nginx --for=condition=Ready pod -l app.kubernetes.io/component=controller --timeout=180s


If you use a locally built app image, load it into the cluster:

# example: kind load docker-image spridoloba/todoapp:2.1.0 --name todoapp
kind load docker-image <your-image:tag> --name todoapp

2) Bootstrap & deploy
chmod +x ./bootstrap.sh
./bootstrap.sh                  # must include build/tag (if needed) and kubectl apply steps

# Or apply manifests explicitly if bootstrap.sh already built images:
kubectl apply -f infrastructure/

3) Validate
kubectl get pods
kubectl get svc
kubectl get ingress


All pods should be Running, and the Ingress should point to todoapp-service on port 80.

4) Access
curl http://localhost
# or open http://localhost in the browser


Pass criteria: application responds successfully at http://localhost
.