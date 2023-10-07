```yaml

```

# Triển khai Keylcoak phiên bản mới nhất trên Microk8s

# Bước 1: Tạo tiệp service.yaml

Chúng ta cần tạo tệp dịch vụ để hiển thị ứng dụng Keycloak. Theo mặc định, Keycloak hoạt động trên cổng 8080, với phiên bản trước, chúng tôi có thể sử dụng trực tiếp ClusterIP và có thể có bản sao của các nhóm. Nhưng với phiên bản keycloak mới nhất, chúng tôi sẽ cần tạo dịch vụ headless, điều đó sẽ giúp chúng tôi có các bản sao, tức là có HA keycloak trên kubernetes.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: keycloak
  labels:
    app: keycloak
spec:
  ports:
  - name: http
    port: 8080
    targetPort: 8080
  selector:
    app: keycloak
  type: ClusterIP
  clusterIP: None
```

