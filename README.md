# tiphys

Helm Chart Abstraction for 80% of Apps. Standardized for NodeJS, Ruby on Rails, and Django

# Usage

```
helm repo add tiphys https://opszero.github.io/tiphys
helm repo update
helm upgrade --install yieldpay tiphys/tiphys -f ./charts/payroll.yaml --set image.repository=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

# Github Action To Create Helm chart using this repo

```
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - name: Release code in production
      run: |
        aws eks --region us-west-2 update-kubeconfig --name ops-prod
        helm repo add tiphys https://opszero.github.io/tiphys
        helm repo update
        helm upgrade --install opsy \
          tiphys/tiphys \
          -n pump \
          --create-namespace \
          -f ./charts/prod.yml \
          --set defaultImage=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG-api \
          --set apps[0].image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG-js \
```

./stage.yaml

```

# Default image for all the apps. If an image isn't specified this one is used.
defaultImage: nginx:latest

# Override with: https://artifacthub.io/packages/helm/bitnami/redis
redis:
  enabled: true # Enable Redis
  # master:
  #  persistence:
  #    size: 50Gi
  # replica:
  #  persistence:
  #    size: 50Gi

datadog:
  enabled: true # Enable Datadog
  envName: develop
  version: "1.1"

envRaw: # Additional environment variables added.
  - name: DD_AGENT_HOST
    valueFrom:
      fieldRef:
        fieldPath: status.hostIP

externals:
  - name: sitemap
    type: ExternalName
    cname: "some-sitemap.s3.amazon.com"
    ingress:
      hosts:
        - host: somesitemap.shopcanal.com
          paths: ["/(sitemap-.*)"]
          port: 8000
  - name: elasticache
    type: ExternalName
    cname: "elasticache.redis.amazon.com"

apps:
  - name: payroll
    secrets:
      USERNAME: opszero
    service:
      enabled: true
      type: ClusterIP
      ports:
        - name: http
          port: 3000
          protocol: TCP
        - name: com
          port: 4000
          protocol: TCP
      ingress:
        annotations:
          nginx.ingress.kubernetes.io/rewrite-target: /$1
        hosts:
          - host: yieldpayroll.com
            paths: ["/"]
            port: 3000
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
      autoscaling:
        enabled: true
        minReplicas: 2
        maxReplicas: 4
        targetCPUUtilizationPercentage: 75
        targetMemoryUtilizationPercentage: 75
      envRaw:
        - name: DD_AGENT_HOST
          valueFrom:
            fieldRef:
              fieldPath: status.hostIP
    jobs:
      - name: db-migrate
        command: "bundle exec rake db:migrate"
    cronjobs:
      - name: hn
        command: ["bundle", "exec", "rails", "pay_people"]
        schedule: "0 * * * *"
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
  - name: payroll
    image: foobar:latest
    service:
      enabled: true
      type: ClusterIP
      ports:
        - name: http
          port: 3000
          protocol: TCP
      command: ["bundle", "exec", "rails", "server"]
      ingress:
        hosts:
          - host: yieldpayroll.com
            paths: ["/"]
            port: 3000
      healthChecks:
        lifecycle:
          preStop:
            exec:
              command:
                - /bin/sh
                - -c
                - sleep 60
        livenessProbe:
          failureThreshold: 15
          initialDelaySeconds: 120
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30
          httpGet:
            path: /healthcheck
            port: 3000
        readinessProbe:
          failureThreshold: 15
          initialDelaySeconds: 120
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 30
          httpGet:
            path: /healthcheck
            port: 3000
      autoscaling:
        enabled: false
      podAnnotations:
        ad.datadoghq.com/celery.logs: '[{"source": "celery","service": "celery"}]'
    jobs:
      - name: db-migrate
        command: "bundle exec rake db:migrate"
    cronjobs:
      - name: hn
        command: ["bundle", "exec", "rails", "pay_people"]
        schedule: "0 * * * *"

secrets:
  KEY_NAME: "Value"
  KEY_NAME2: "Value"
```

# Helm Template

```
helm template ./charts/tiphys -f ./stage.yaml
```
