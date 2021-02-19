# Helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
# kubeapps
```
helm repo add bitnami https://charts.bitnami.com/bitnami
kubectl create namespace kubeapps
helm install kubeapps --namespace kubeapps bitnami/kubeapps
```
# ingress-nginx
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/cloud/deploy.yaml
```
# ingress http
kubeapps-ingress_http.yaml
---
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kubeapps-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: kubeapps.<domain>
    http:
      paths:
      - backend:
          serviceName: kubeapps
          servicePort: 80
```          
          
## run
```
kubectl apply -f kubeapps-ingress_http.yaml -n kubeapps
```
## access

http://kubeapps.domain


# cert-manager
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.2.0/cert-manager.yaml
```
          
# https
```
kubectl delete -f kubeapps-ingress_http.yaml -n kubeapps		  
```
## Cluster issuer

production_issuer.yaml
--
```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # Email address used for ACME registration
    email: you_email_id@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Name of a secret used to store the ACME account private key
      name: letsencrypt-prod-private-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
```
## run	
```
kubectl create -f production_issuer.yaml	  
```		  
		  
## create ingress resource https

kubeapps-ingress.yaml
---
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kubeapps-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - kubeapps.<domain>
    secretName: kubeapps-cert-tls
  rules:
  - host: kubeapps.<domain>
    http:
      paths:
      - backend:
          serviceName: kubeapps
          servicePort: 80
```
## run

```
kubectl apply -f kubeapps-ingress.yaml -n kubeapps

kubectl create serviceaccount kubeapps-operator
kubectl create clusterrolebinding kubeapps-operator --clusterrole=cluster-admin --serviceaccount=default:kubeapps-operator
```

## get token
```
kubectl get secret $(kubectl get serviceaccount kubeapps-operator -o jsonpath='{range .secrets[*]}{.name}{"\n"}{end}' | grep kubeapps-operator-token) -o jsonpath='{.data.token}' -o go-template='{{.data.token | base64decode}}' && echo
```

## access kubeapps
https://kubeapps.<domain>

give token and login


# wordpress		  

## install wordpress in kubeapps

Create namespace wordpress

Then deploy

give name mywordpress

## ingress

wordpress-ingress.yaml
--
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kubeapps-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - wordpress.<domain>
    secretName: wordpress-cert-tls
  rules:
  - host: wordpress.<domain>
    http:
      paths:
      - backend:
          serviceName: mywordpress
          servicePort: 80
        path: /
        pathType: ImplementationSpecific
```
## run
```
kubectl apply -f wordpress-ingress.yaml  -n wordpress
```
## access

https://wordpress.domain

## cleanup wordpress

Delete the wordpress in kubeapps

kubectl delete namespace wordpress

kubectl delete -f wordpress-ingress.yaml -n wordpress 

## cleanup kubeapps (we will need it in later videos,so you should keep it)
```
helm uninstall kubeapps --namespace kubeapps
kubectl delete clusterrolebinding kubeapps-operator
kubectl delete serviceaccount kubeapps-operator
kubectl delete namespace kubeapps
```
