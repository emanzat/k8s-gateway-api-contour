# k8s-gateway-api-contour
![image](https://github.com/user-attachments/assets/15b4e88f-d870-4012-b72e-53a9b8e86236)


Install Dynamically provisioned

kubectl apply -f https://projectcontour.io/quickstart/contour-gateway-provisioner.yaml

kubectl apply -f - <<EOF
kind: GatewayClass
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: contour
spec:
  controllerName: projectcontour.io/gateway-controller
EOF

Create a Gateway:
kubectl apply -f - <<EOF
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
EOF
