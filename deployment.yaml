apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: main-container
          image: my-app-image
          # Your main container configuration goes here
      containers:
        - name: vault-agent
          image: hashicorp/vault-k8s:latest
          args:
            - "agent"
            - "-config=/vault/config/agent-config.hcl"
          volumeMounts:
            - name: agent-config
              mountPath: /vault/config
        - name: sidecar
          image: busybox
          command: ["sh", "-c", "cp /vault/secrets/my-secret /app/secrets/my-secret"]
          volumeMounts:
            - name: app-secrets
              mountPath: /app/secrets
          volumeMounts:
            - name: vault-token
              mountPath: /var/run/secrets/vault
      volumes:
        - name: agent-config
          configMap:
            name: vault-agent-config
        - name: app-secrets
          emptyDir: {}
        - name: vault-token
          projected:
            sources:
              - secret:
                  name: vault-token
                  items:
                    - key: token
                      path: token
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: vault-agent-config
data:
  agent-config.hcl: |
    auto_auth {
      method "kubernetes" {
        mount_path = "auth/kubernetes"
        config = {
          role = "my-app-role"
        }
      }

      sink "file" {
        config = {
          path = "/var/run/secrets/vaultproject.io/vault-agent"
        }
      }
    }

    template {
      destination = "/vault/secrets"
      contents = {
        source = "/app/secrets"
      }
    }
---
apiVersion: v1
kind: Secret
metadata:
  name: vault-token
stringData:
  token: <Your_Vault_Auth_Token>
