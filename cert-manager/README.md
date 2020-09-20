# Steps to install cert-manager on k3s

Given that the first version installed was v0.13.1 using manual install of CRDs I took the decision to wipe the cert-management installation and install from zero. As my install doesn't need to be high availability and can survive few minutes of ingress downtime I will include here the steps to clean and old install and install new one.

## Find and remove all ingress objects

Find and delete all the ingres objects from the `kubectl get ingress -A` command and remember to add them back using the manifests. If you don't have manifests, export them into a folder so you can add them back.

## Helm uninstall cert-manager

```
$ helm uninstall --namespace cert-manager cert-manager-1583609748
release "cert-manager-1583609748" uninstalled
```

## Delete namespace to clean it further

```
$ kubectl delete ns cert-manager
```

## Delete the CRDs

```
$ kubectl delete -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.1/deploy/manifests/00-crds.yaml
customresourcedefinition.apiextensions.k8s.io "certificaterequests.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "certificates.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "challenges.acme.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "clusterissuers.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "issuers.cert-manager.io" deleted
customresourcedefinition.apiextensions.k8s.io "orders.acme.cert-manager.io" deleted
```

## Create namespace

```
$ kubectl create namespace cert-manager
namespace/cert-manager created
```

## Add the Jetstack Helm repository and update repo

```
$ helm repo add jetstack https://charts.jetstack.io
"jetstack" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jetstack" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

## Install cert-manager using helm

```
$ helm install cert-manager jetstack/cert-manager --namespace cert-manager \
  --version v0.16.1 --set installCRDs=true
NAME: cert-manager
LAST DEPLOYED: Sat Aug 15 14:49:36 2020
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

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
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

## Add back all the ingress
