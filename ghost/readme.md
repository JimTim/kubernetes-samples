```yaml
apiVersion: v1
stringData:
  from: 
  host: 
  port: 
  username: 
  password: 
kind: Secret
metadata:
  labels:
    app.kubernetes.io/name: ghost
  name: ghost-mail-secret
  namespace: ghost
type: Opaque
```