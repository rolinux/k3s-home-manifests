# Steps to install cert-manager on k3s

## Create CustomResourceDefinition (CRD)

```
# kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.1/deploy/manifests/00-crds.yaml
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
```

## Create namespace

```
# kubectl create namespace cert-manager
namespace/cert-manager created
```

## Add the Jetstack Helm repository and update repo

```
# helm repo add jetstack https://charts.jetstack.io
"jetstack" has been added to your repositories

# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jetstack" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

## Install cert-manager using helm

```
# helm install --namespace cert-manager --version v0.13.1  --generate-name jetstack/cert-manager
NAME: cert-manager-1580422767
LAST DEPLOYED: Thu Jan 30 22:19:31 2020
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://docs.cert-manager.io/en/latest/reference/issuers.html

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://docs.cert-manager.io/en/latest/reference/ingress-shim.html
```

## Generate a signing key pair (CA)

Generate a CA private key

```
$ openssl genrsa -out ca.key 2048
```

Create a self signed Certificate, valid for 10yrs with the 'signing' option set

```
$ openssl req -x509 -new -nodes -key ca.key -subj "/CN=h.coroi.net" -days 3650 -reqexts v3_req -extensions v3_ca -out ca.crt
```

## Create CA secret

```
kubectl create secret tls ca-key-pair \
   --cert=ca.crt \
   --key=ca.key \
   --namespace=cert-manager
```

## Create ClusterIssuer

```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: ca-issuer
spec:
  ca:
    secretName: ca-key-pair
```
