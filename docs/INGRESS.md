# Ingress

All ingress is handled by the [nginx ingress controller](https://kubernetes.github.io/ingress-nginx/) which is 
maintained under the [kubernetes github organization](https://github.com/kubernetes/ingress-nginx/). 
By default traffic that does not match any ingress will receive the mvpstudio.org homepage.

SSL Certs are handled automatically by [cert manager](https://cert-manager.io/) on any `Ingress` resource with the 
`cert-manager.io/cluster-issuer: cluster-certs` annotation.

There is a cluster wide [ClusterIssuer](../running/cert-manager/cluster-issuer.yaml) so any namespace can issue their 
necessary SSL certs. It uses a HTTP01 challenge so as longas the DNS hostname is directed to the public ip address of 
the kubernetes cluster (67.59.93.161) then the cert will setup automatically.

This is an example of creating an ingress for a project that will match all traffic for the hostnames 
`helloworld.prod.apps.mvpstudio.org` and `helloworld.mvpstudio.org` and route all that traffic to a single web server.
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world
  namespace: hello-world
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: cluster-certs
spec:
  tls:
    - hosts:
        - helloworld.prod.apps.mvpstudio.org
        - helloworld.mvpstudio.org
      secretName: hello-world-cert
  rules:
    - host: helloworld.prod.apps.mvpstudio.org
      http:
        paths:
          - path: /
            backend:
              serviceName: web
              servicePort: 80
    - host: helloworld.mvpstudio.org
      http:
        paths:
          - path: /
            backend:
              serviceName: web
              servicePort: 80
```

## Debugging

### Nginx Ingress

The ingress nginx controller scales horizontally a little differently then other controllers. Sometimes projects will 
have a single controller that spins up other pods or resources. In this case the controller is a deployment, and each 
pod watches k8 resources and updates its own config. So to scale the ingress pods we can simply add more replicas to 
the deployment. It is current set to 3 so one runs on each worker node. 

Check the [documented troubleshooting page](https://kubernetes.github.io/ingress-nginx/troubleshooting/).

The logging has been turned up to level 3, so using kubectl to get logs of a nginx controller pod will show any 
resource changes and events for any ingress resource.

Printing the nginx config from a controller can also be useful:
```bash
kubectl exec -it -n ingress-nginx ingress-nginx-controller-<id> -- cat /etc/nginx/nginx.conf
```

### Cert Manager

Check the ACME & HTTP01 [troubleshooting guide](https://cert-manager.io/docs/faq/acme/). We are using ACME with lets 
encrypt using an HTTP01 challenge.

It's worth doing a `kubectl describe` on any `certificate`, `certificaterequest`, and `order` resources within the 
namespace of the project.
