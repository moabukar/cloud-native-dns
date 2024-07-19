# Cloud-native DNS setup (cert-manager, externalDNS, nginx ingress controller)

### Prerequisites:

- AKS or EKS cluster or even GKE

## NGINX Ingress controller

- This will create a K8s service with a LB type and thus create a LB in your chosen cloud with a publicIP. This will be or SHOULD be the route for your cluster apps

- Install an nginx ingress controller on your cluster (make sure you're using a cloud cluster, local clusters will not work unless you have metalLB installed)

## ExternalDNS

- Setup externalDNS below. We will use Cloudflare here to demonstrate. You need to create a Cloudflare token that has 2 perms:
    - Zone:Read
    - DNS:Edit
- Use this token to create a secret (as shown below)
- Install Helm and start playing.

ExternalDNS

```bash
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm repo update

kubectl create ns external-dns

kubectl create secret generic cloudflare-api-key --from-literal=apiKey="<>" --namespace external-dns

helm upgrade --install external-dns external-dns/external-dns --values values.yml --namespace external-dns
```

## Cert-manager

- Create an issuer and use this for all cert generation and challenges. It will create Let's encrypt certs for you


### Final steps

- Create an `A` record in your cloudlfare settings with name `@` to point to itself which will be your domain name and in my case `moabukar.co.uk` with the value of your ingress controller IP. You can get this by doing `kubectl get services -n ingress-nginx` or wherever your svc for nginx ingress controller is deployed. Get the externalIP which is the public on and put this on Cloudflare.

- Create a test deployment and a service with clusterIP
- Create an ingress resource with your chosen subdomain and domain which you own
    - in my case `home.moabukar.co.uk`.
    - `home` being my subdomain
    - `moabukar.co.uk` being by own domain

- externalDNS and cert-manager will do the magic of automation. They will automatically add the DNS records required into your DNS settings on cloudlfare and cert-manager will create a Let's encrypt cert for you with TLS so you can access your app via HTTPS. 

so I can access my homepage on `https://home.moabukar.co.uk`

Anddd that's it!!
