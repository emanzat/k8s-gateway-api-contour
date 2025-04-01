# k8s-gateway-api-contour
![image](https://github.com/user-attachments/assets/15b4e88f-d870-4012-b72e-53a9b8e86236)

# Introduction

Dans notre article précédent, nous avons largement couvert l'API Gateway de Kubernetes, en mettant l'accent sur des implémentations clés telles que le routage HTTP, le routage basé sur les en-têtes, la redirection HTTP et la réécriture des routes. En nous appuyant sur ces bases, nous allons maintenant explorer d'autres cas d'utilisation afin d'approfondir notre compréhension de cet outil puissant. Suivez-nous pour découvrir des scénarios concrets illustrant la polyvalence et la praticité de l'API Gateway de Kubernetes.

# Scénarios documentés

Nous allons documenter les scénarios suivants :

- Répartition du trafic HTTP
- Routage inter-namespaces
- TLS

## 1. Répartition du trafic HTTP

La ressource `HTTPRoute` permet de répartir le trafic entre plusieurs services backend en fonction de poids définis. Cette fonctionnalité est utile pour gérer les déploiements progressifs, le canary testing ou les situations d'urgence.

### Exemple de configuration :

```yaml
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
    name: deploy
    namespace: projectcontour
    labels:
        app: deploy
spec:
    parentRefs:
        - group: gateway.networking.k8s.io
          kind: Gateway
          name: contour
    hostnames:
        - "local.projectcontour.io"
    rules:
        - matches:
          - path:
              type: PathPrefix
              value: "/"
          backendRefs:
          - kind: Service
            name: green
            port: 80
            weight: 70
          - kind: Service
            name: blue
            port: 80
            weight: 30
```

### Test de la répartition du trafic :

```sh
kubectl -n projectcontour port-forward service/envoy 8888:80
curl -i http://local.projectcontour.io:8888
```

## 2. Routage inter-namespaces

L'API Gateway prend en charge le routage entre namespaces. Cette fonctionnalité est utile lorsque plusieurs équipes partagent la même infrastructure réseau tout en conservant une séparation des accès et configurations.

### Exemple de configuration :

```yaml
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
    name: contour
    namespace: projectcontour
spec:
    gatewayClassName: contour
    listeners:
      - name: http
        protocol: HTTP
        port: 80
        allowedRoutes:
          namespaces:
            from: All
```

### Test du routage inter-namespaces :

```sh
kubectl -n projectcontour port-forward service/envoy 8888:80
curl -i http://local.projectcontour.io:8888
```

## 3. TLS

L'API Gateway permet de gérer indépendamment la configuration TLS des connexions en amont et en aval.

### Création d'un certificat TLS auto-signé :

```sh
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -days 365 -key ca.key -out ca.crt -subj "/CN=yourdomain.com"
kubectl create secret tls ca-tls-secret --key ca.key --cert ca.crt
```

### Configuration d'un Gateway avec TLS :

```yaml
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
    name: contour
    namespace: projectcontour
spec:
    gatewayClassName: contour
    listeners:
        - name: http
          protocol: HTTP
          port: 80
        - name: https
          protocol: HTTPS
          port: 443
          hostname: "local.projectcontour.io"
          tls:
            certificateRefs:
              - kind: Secret
                name: ca-tls-secret
                namespace: projectcontour
```

### Test de la configuration TLS :

```sh
curl --resolve local.projectcontour.io:$GW_IP:$GW_PORT https://local.projectcontour.io:$GW_PORT/ --insecure
```

# Conclusion

L'API Gateway de Kubernetes offre de puissantes fonctionnalités pour gérer le routage HTTP, la répartition du trafic, le routage inter-namespaces et le chiffrement TLS. Ces scénarios démontrent la flexibilité et la robustesse de cette solution dans des environnements Kubernetes complexes.
