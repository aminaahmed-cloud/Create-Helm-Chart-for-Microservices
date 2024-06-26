# Create-Helm-Chart-for-Microservices

This guide walks through the process of creating Helm charts for deploying microservices on a Kubernetes cluster. We'll follow a blueprint for Deployment and Service for all microservices and set values for individual microservices. Additionally, we'll use a combination of shared charts for similar applications and separate charts for completely different apps.

---

## Basic Structure of Helm Chart

### Create Shared Helm Chart

```bash
helm create microservice
```

This command creates the basic directory structure. I'll write all the templates I need from scratch, so I'll delete the autogenerated files

<img src="https://i.imgur.com/c8KeeFB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

### Create Microservices Helm Chart: Create Basic Template File

I'll begin by creating basic template files for Deployment and Service. Please note that I have to specify values.

**- Values Object:**
  - A built-in object.
  - By default, values are empty.
**- Values are passed into the template from 3 sources:**
    - The values.yaml file in the chart.
    -  User-supplied file passed with -f flag.
    -  Parameter passed with --set flag.
**- Variable Naming Conventions:**
  - Names should begin with a lowercase letter.
  - Separated with camel case.

**Dynamic Environment Variables**
I'll use Range to provide a "for each"-style loop to iterate through or "range over" a list.
I'll also use Pipelines, which have the same concept as UNIX and are a tool for chaining together template commands.

---

**deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
spec:
  replicas: {{ .Values.appReplicas }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: {{ .Values.appName }}
        image: "{{ .Values.appImage }}:{{ .Values.appVersion }}"
        ports:
        - containerPort: {{ .Values.containerPort }}
        env:
        {{- range .Values.containerEnvVars}}
        - name: {{ .name }}
          value: {{ .value | quote }}
        {{- end}}
```

**service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
spec:
  type: {{ .Values.serviceType }}
  selector:
    app: {{ .Values.appName }}
  ports:
  - protocol: TCP
    port: {{ .Values.servicePort }}
    targetPort: {{ .Values.containerPort }}
```

**values.yaml**

```yaml
appName: servicename
appImage: gcr.io/google-samples/microservices-demo/servicename
appVersion: v0.0.0
appReplicas: 1
containerPort: 8080
containerEnvVars: 
- name: ENV_VAR_ONE
  value: "valueone"
- name: ENV_VAR_TWO
  value: "valuetwo"

servicePort: 8080
serviceType: ClusterIP
```

---
### Create Microservices Helm Chart: Create Values File for All

I'll start by creating separate values files for each microservice.

**email-service-values.yaml**

```yaml

appName: emailservice
appImage: gcr.io/google-samples/microservices-demo/emailservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 8080
containerEnvVars: 
- name: PORT
  value: "8080"

servicePort: 5000
```

**Deploy Service**

```bash
helm template -f email-service-values.yaml /Users/ahmed/k8s-configuration/microservices/microservice
```

```bash
helm install -f values/email-service-values.yaml emailservice microservice
```

<img src="https://i.imgur.com/09I5vwW.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

**ad-service-values.yaml**

```yaml
appName: adservice
appImage: gcr.io/google-samples/microservices-demo/adservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 9555
containerEnvVars: 
- name: PORT
  value: "9555"

servicePort: 9555
```

**Deploy Service**

```bash
helm install -f values/ad-service-values.yaml adservice microservice
```

---

**cart-service-values.yaml**

```yaml
appName: cartservice
appImage: gcr.io/google-samples/microservices-demo/cartservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 7070
containerEnvVars: 
- name: PORT
  value: "7070"
- name: REDIS_ADDR
  value: "redis-cart:6379"

servicePort: 7070
```

**Deploy Service**

```bash
helm install -f values/cart-service-values.yaml cartservice microservice
```

---

**checkout-service-values.yaml**

```yaml
appName: checkoutservice
appImage: gcr.io/google-samples/microservices-demo/checkoutservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 5050
containerEnvVars: 
  - name: PORT
    value: "5050"
  - name: PRODUCT_CATALOG_SERVICE_ADDR   
    value: "productcatalogservice:3550"
  - name: SHIPPING_SERVICE_ADDR
    value: "shippingservice:50051"
  - name: PAYMENT_SERVICE_ADDR
    value: "paymentservice:50051"    
  - name: EMAIL_SERVICE_ADDR
    value: "emailservice:5000"
  - name: CURRENCY_SERVICE_ADDR
    value: "currencyservice:7000"
  - name: CART_SERVICE_ADDR
    value: "cartservice:7070"

servicePort: 5050
```

**Deploy Service**

```bash
helm install -f values/checkout-service-values.yaml checkoutservice microservice
```

---

**currency-service-values.yaml**

```yaml
appName: currencyservice
appImage: gcr.io/google-samples/microservices-demo/currencyservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 7000
containerEnvVars: 
- name: PORT
  value: "7000"
- name: DISABLE_PROFILER
  value: "1"

servicePort: 7000
```

**Deploy Service**

```bash
helm install -f values/currency-service-values.yaml currencyservice microservice
```

---

**payment-service-values.yaml**

```yaml
appName: paymentservice
appImage: gcr.io/google-samples/microservices-demo/paymentservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 50051
containerEnvVars: 
- name: PORT
  value: "50051"
- name: DISABLE_PROFILER
  value: "1"

servicePort: 50051
```

**Deploy Service**

```bash
helm install -f values/payment-service-values.yaml paymentservice microservice
```

---

**productcatalog-service-values.yaml**

```yaml
appName: productcatalogservice
appImage: gcr.io/google-samples/microservices-demo/productcatalogservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 3550
containerEnvVars: 
- name: PORT
  value: "3550"
- name: DISABLE_PROFILER
  value: "1"

servicePort: 3550
```

**Deploy Service**

```
helm install -f values/productcatalog-service-values.yaml productcatalogservice microservice
```

---

**recommendation-service-values.yaml**

```yaml
appName: recommendationservice
appImage: gcr.io/google-samples/microservices-demo/recommendationservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 8080
containerEnvVars: 
- name: PORT
  value: "8080"
- name: PRODUCT_CATALOG_SERVICE_ADDR
  value: "productcatalogservice:3550"

servicePort: 8080
```

**Deploy Service**

```
helm install -f values/recommendation-service-values.yaml recommendationservice microservice
```

---

**shipping-service-values.yaml**

```yaml
appName: shippingservice
appImage: gcr.io/google-samples/microservices-demo/shippingservice
appVersion: v0.8.0
appReplicas: 2
containerPort: 50051
containerEnvVars: 
- name: PORT
  value: "50051"

servicePort: 50051
```

**Deploy Service**

```
helm install -f values/shipping-service-values.yaml shippingservice microservice
```

---

**frontend-values.yaml**

```yaml
appName: frontend
appImage: gcr.io/google-samples/microservices-demo/frontend
appVersion: v0.8.0
appReplicas: 2
containerPort: 8080
containerEnvVars: 
- name: PORT
  value: "8080"
- name: PRODUCT_CATALOG_SERVICE_ADDR
  value: "productcatalogservice:3550"
- name: CURRENCY_SERVICE_ADDR
  value: "currencyservice:7000"
- name: CART_SERVICE_ADDR
  value: "cartservice:7070"
- name: RECOMMENDATION_SERVICE_ADDR
  value: "recommendationservice:8080"
- name: SHIPPING_SERVICE_ADDR
  value: "shippingservice:50051"
- name: CHECKOUT_SERVICE_ADDR
  value: "checkoutservice:5050"
- name: AD_SERVICE_ADDR
  value: "adservice:955"

servicePort: 80
serviceType: LoadBalancer
```

**Deploy Service**

```
helm install -f values/frontend-values.yaml frontend microservice
```


<img src="https://i.imgur.com/sTSNJ32.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


---

<img src="https://i.imgur.com/YnhhxI2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


---

### Create Redis Helm Chart

```bash
helm create redis
```
**Deployment Configuration (deployment.yaml)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
spec:
  replicas: {{ .Values.appReplicas }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: {{ .Values.appName }}
        image: "{{ .Values.appImage }}:{{ .Values.appVersion }}"
        ports:
        - containerPort: {{ .Values.containerPort }}
        livenessProbe:
          initialDelaySeconds: 5
          tcpSocket:
            port: {{ .Values.containerPort }}
          periodSeconds: 5
        readinessProbe:
          initialDelaySeconds: 5
          tcpSocket:
            port: {{ .Values.containerPort }}
          periodSeconds: 5
        resources:
          requests: 
            cpu: 70m
            memory: 200Mi
          limits:
            cpu: 125m
            memory: 300Mi
        volumeMounts:
        - name: {{ .Values.volumeName }}
          mountPath: {{ .Values.containerMountPath }}
      volumes:
      - name: {{ .Values.volumeName }}
        emptyDir: {}
```

---

**Service Configuration (service.yaml)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
spec:
  type: ClusterIP
  selector:
    app: {{ .Values.appName }}
  ports:
  - protocol: TCP
    port: {{ .Values.servicePort }}
    targetPort: {{ .Values.containerPort }} 
```
---

**Redis Values Configuration (redis-values.yaml)**

```yaml
appName: redis-cart
appReplicas: 2
```

**Deploy Service**

```bash
helm install -f values/redis-values.yaml redis-cart microservice
```

