# Запуск ComfyUI сервера в кубе

Глобальные переменные:

```sh
COMFY_UI_NS=hudojka

COMFY_UI_TAG="cuda12-1771003088"
COMFY_UI_IMAGE="docker.io/78945789345654/comfyui:$COMFY_UI_TAG"

COMFY_NODE_HOSTNAME="<secret>"
COMFY_SVC_PORT="80"

# это путь внутри файловой системы пода
COMFY_UI_CUSTOM_MODELS_PATH_INSIDE_POD=/usr/local/comfyui/downloaded_models
```

Отдельный неймспейсик:

```sh
kubectl apply --filename - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: $COMFY_UI_NS
EOF
```

Конфиг для ManagerUI:

```sh
kubectl apply --filename - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: comfyui-manager-conf
  namespace: $COMFY_UI_NS
data:
  config.ini: |
    [default]
    git_exe = 
    use_uv = True
    channel_url = https://raw.githubusercontent.com/ltdrdata/ComfyUI-Manager/main
    share_option = all
    bypass_ssl = False
    file_logging = True
    update_policy = stable-comfyui
    windows_selector_event_loop_policy = False
    model_download_by_agent = False
    downgrade_blacklist = 
    security_level = normal
    always_lazy_install = False
    network_mode = personal_cloud
    db_mode = cache
    verbose = True
EOF
```

Конфиг для дополнительных путей с моделями в ComfyUI:

```sh
kubectl apply --filename - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: comfyui-extra-models-conf
  namespace: $COMFY_UI_NS
data:
  extra_model_paths.yaml: |
    shared_models:
      base_path: $COMFY_UI_CUSTOM_MODELS_PATH_INSIDE_POD
      loras: loras
      checkpoints: checkpoints
      vae: vae
      controlnet: controlnet
      upscale_models: upscale_models
      clip: clip
      diffusion_models: diffusion_models
EOF
```

Деплоймент с ComfyUI сервером и node affinity рулами:

```sh
kubectl apply --filename - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: comfyui
  namespace: $COMFY_UI_NS
spec:
  selector:
    matchLabels:
      app: comfyui
  template:
    metadata:
      labels:
        app: comfyui
    spec:
      containers:
      - name: comfyui
        image: $COMFY_UI_IMAGE
        command:
        - python
        args:
        - main.py
        - --listen
        - 0.0.0.0
        - --port
        - "8188"
        - --enable-manager
        ports:
        - containerPort: 8188
          name: comfyui-http
        volumeMounts:
        - name: custom-nodes
          mountPath: /usr/local/comfyui/custom_nodes
        - name: downloaded-models
          mountPath: $COMFY_UI_CUSTOM_MODELS_PATH_INSIDE_POD
        - name: manager-ui-conf
          # The specific file path inside the container
          mountPath: /usr/local/comfyui/user/__manager/config.ini
          # The key name from the ConfigMap
          subPath: config.ini
        - name: extra-models-conf
          # The specific file path inside the container
          mountPath: /usr/local/comfyui/extra_model_paths.yaml
          # The key name from the ConfigMap
          subPath: extra_model_paths.yaml
        resources:
          requests:
            nvidia.com/gpu: 1
          limits:
            nvidia.com/gpu: 1
      volumes:
      - name: custom-nodes
        hostPath:
          # создается автоматически, владелец root
          path: /usr/local/hudojka/custom_nodes
      - name: manager-ui-conf
        configMap:
          name: comfyui-manager-conf
      - name: downloaded-models
        hostPath:
          # создается автоматически, владелец root
          path: /usr/local/hudojka/downloaded_models
      - name: extra-models-conf
        configMap:
          name: comfyui-extra-models-conf
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: nvidia.com/gpu.product
                operator: In
                values:
                - NVIDIA-A100-80GB-PCIe
EOF
```

Деплоймент с ComfyUI сервером и node selector рулами:

```sh
kubectl apply --filename - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: comfyui
  namespace: $COMFY_UI_NS
spec:
  selector:
    matchLabels:
      app: comfyui
  template:
    metadata:
      labels:
        app: comfyui
    spec:
      containers:
      - name: comfyui
        image: $COMFY_UI_IMAGE
        command:
        - python
        args:
        - main.py
        - --listen
        - 0.0.0.0
        - --port
        - "8188"
        - --enable-manager
        ports:
        - containerPort: 8188
          name: comfyui-http
        volumeMounts:
        - name: custom-nodes
          mountPath: /usr/local/comfyui/custom_nodes
        - name: downloaded-models
          mountPath: $COMFY_UI_CUSTOM_MODELS_PATH_INSIDE_POD
        - name: manager-ui-conf
          # The specific file path inside the container
          mountPath: /usr/local/comfyui/user/__manager/config.ini
          # The key name from the ConfigMap
          subPath: config.ini
        - name: extra-models-conf
          # The specific file path inside the container
          mountPath: /usr/local/comfyui/extra_model_paths.yaml
          # The key name from the ConfigMap
          subPath: extra_model_paths.yaml
        resources:
          requests:
            nvidia.com/gpu: 1
          limits:
            nvidia.com/gpu: 1
      volumes:
      - name: custom-nodes
        hostPath:
          # создается автоматически, владелец root
          path: /usr/local/hudojka/custom_nodes
      - name: manager-ui-conf
        configMap:
          name: comfyui-manager-conf
      - name: downloaded-models
        hostPath:
          # создается автоматически, владелец root
          path: /usr/local/hudojka/downloaded_models
      - name: extra-models-conf
        configMap:
          name: comfyui-extra-models-conf
      nodeSelector:
        kubernetes.io/hostname: $COMFY_NODE_HOSTNAME
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: nvidia.com/gpu.product
                operator: In
                values:
                - Tesla-V100-PCIE-16GB
EOF
```

Внутренний сервис:

```sh
kubectl apply --filename - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: comfyui
  namespace: $COMFY_UI_NS
spec:
  type: ClusterIP
  ports:
  - name: http
    port: $COMFY_SVC_PORT
    protocol: TCP
    targetPort: comfyui-http
  selector:
    app: comfyui
EOF
```

Настройка ингресса для доступа извне (Istio):

```sh
ISTIO_ROOT_NS='istio-system'
DOMAIN='<secret>'

kubectl apply --filename - <<EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: comfyui-gateway-cert
  namespace: $ISTIO_ROOT_NS
spec:
  secretName: comfyui-gateway-cert
  issuerRef:
    kind: Issuer
    name: letsencrypt
  commonName: comfyui.$DOMAIN
  dnsNames:
  - comfyui.$DOMAIN
---
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  name: comfyui-gateway
  namespace: $COMFY_UI_NS
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      name: https
      number: 443
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: comfyui-gateway-cert
    hosts:
    - comfyui.$DOMAIN
---
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: comfyui-virt-svc
  namespace: $COMFY_UI_NS
spec:
  gateways:
  - comfyui-gateway
  hosts:
  - comfyui.$DOMAIN
  http:
  - route:
    - destination:
        host: comfyui.$COMFY_UI_NS.svc.cluster.local
        port:
          number: $COMFY_SVC_PORT
EOF
```

Настройка ингресса для доступа извне (ingress-nginx):

```sh
COMFY_UI_NS='hudojka'
APELSIN_DOMAIN='<secret>'

kubectl apply -f - << EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: comfyui-ingress
  namespace: $COMFY_UI_NS
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-nginx"
    kubernetes.io/tls-acme: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - comfyui.$APELSIN_DOMAIN
    secretName: comfyui-ingress-cert
  rules:
  - host: comfyui.$APELSIN_DOMAIN
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: comfyui
            port:
              name: http
EOF
```
