# Efucloud Community Resources

- 中文快速入口： [README.zh.md](README.zh.md)

## Overview
### kube-keeper
kube-keeper is an AI + cloud-native distributed all-in-one R&D, delivery, and operations platform built on Kubernetes. It supports multi-tenant isolation and unified public/private cloud management. It provides LEGO-style visual application orchestration, AI-assisted operations, visual CI/CD pipelines, a private app marketplace, an MCP plaza, and an integrated distributed cloud IDE for secure and efficient end-to-end development, delivery, and operations. This repository provides the single-edition version.
This project does not include built-in authentication and requires integration with the open-source EAuth.

### EAuth
EAuth is an enterprise-grade authentication platform that offers multiple authentication methods, including face recognition, OpenID Connect (OIDC), and web authentication. It supports multi-factor authentication (MFA) and flexible token generation policies to enable single sign-on across applications, helping enterprises build fine-grained authentication management and improve authentication efficiency.

## EAuth
### Source code
1. Frontend: [github.com/efucloud/eauth](https://github.com/efucloud/eauth)
2. Backend: [github.com/efucloud/eauth-console](https://github.com/efucloud/eauth-console)

### Deployment
1. Create the `efucloud` namespace:
```sh
kubectl apply -f namespace.yaml
```
2. Deploy MySQL if you do not already have a database service:
```sh
# For persistence, uncomment/comment the PVC and mount sections inside the manifest.
kubectl apply -f mysql.yaml
```
3. Update `backend.yaml` with your environment values. Example secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: eauth-config
  namespace: efucloud
type: Opaque
stringData:
  config.yaml: |
    # Serves as the base address that appears in /.well-known/openid-configuration.
    serverAddress: "http://eauth-demo.efucloud.com"
    tokenPeriod: 16
    # User avatar upload path; mount a PVC for persistence.
    uploadPath: "/efucloud/uploads"
    # Login method configuration
    loginConfig:
      # Enable face recognition
      faceRecognition: true
      # Enable MFA (global switch). Once enabled, disabling later does not remove MFA for already added users.
      mfa: false
    # Database configuration
    mysql:
      host: "mysql:3306"
      user: "root"
      password: "EfuCloudPSD"
      dbname: "eauth"
      charset: "utf8mb4"
      loc: "Asia/Shanghai"
      defaultStringSize: 0
      disableDatetimePrecision: false
      dontSupportRenameColumn: false
      dontSupportRenameIndex: false
      skipInitializeWithVersion: false
    # Email settings used for password recovery
    email:
      smtpServer: "smtp.qq.com"
      smtpPort: 465
      username: "noreply@example.com"
      password: "CHANGE_ME"
    logConfig:
      production: true
      filename: ""
      maxsize: 100
      maxbackups: 7
      maxage: 30
      compress: true
      localtime: true
```
4. Deploy the backend and frontend. Adjust the Ingress settings in `frontend.yaml` if you use a custom domain, and add TLS configuration when possible.
```sh
kubectl apply -f backend.yaml
kubectl apply -f frontend.yaml
```

## kube-keeper deployment
### Notes
kube-keeper does not ship with its own authentication; connect it to EAuth for login management.

### Deployment steps
1. Create the namespace:
```sh
kubectl create ns efucloud
```
2. Create the secret for configuration. First register an application in EAuth and capture the `clientId`, `clientSecret`, and the callback URL (`https://<your-kubekeeper-domain>/oauth/callback`):
```yaml
kind: Secret
apiVersion: v1
metadata:
  name: kube-keeper-config
  namespace: efucloud
type: Opaque
stringData:
  config.yaml: |
    databaseAutoMigrate: true
    logConfig:
      compress: false
      filename: ""
      level: ""
      localtime: false
      maxage: 0
      maxbackups: 0
      maxsize: 0
      production: true
    mysql:
      charset: utf8
      dbname: kubekeeper
      defaultStringSize: 0
      disableDatetimePrecision: false
      dontSupportRenameColumn: false
      dontSupportRenameIndex: false
      host: kubekeeper-mysql:3306
      loc: ""
      password: EfuCloud@Pwd
      skipInitializeWithVersion: false
      user: root
    # EAuth authentication configuration
    oidcConfig:
      clientId: fvqahqk3vr3hsqguhes2u6nff
      clientSecret: gr6vfm63g6ndhythkx44p3aamfs4k3nfumd5qz24ofpdsr5747q
      issuer: http://eauth-demo.efucloud.com
    # Third-party LLM API integration
    chatConfig:
      useTool: true
      address: https://dashscope.aliyuncs.com/compatible-mode/v1
      apiKey: sk-23486e1c9ed44d45cbb4babeaa
      model: qwen3-coder-480b-a35b-instruct
    # Administrator email addresses; applied when users sign in
    adminEmails:
      - admin@efucloud.cn
      - admin@efucloud.com
    # Container image used by terminal sessions
    terminalContainer: registry.cn-shenzhen.aliyuncs.com/efucloud-public/k8s-tools:v1.0.0.03132013
```
3. Apply the deployment manifest:
```sh
kubectl apply -f deployment.yaml
```
4. Access kube-keeper via port-forward if you run in Kind or Minikube:
```sh
kubectl port-forward -n efucloud svc/<service-name> 8080:8080
```

### Tutorials
Check `docs/eauth/images/eauth.png` for the OIDC configuration screenshot and consult the Chinese README for additional guides and video links.
