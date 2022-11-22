# tiphys

Standardized App Deployments for Node, Ruby on Rails, and Django apps

# Usage

```
helm repo add tiphys https://opszero.github.io/tiphys
helm repo update
helm upgrade --install yieldpay tiphys/tiphys -f ./charts/payroll.yaml --set image.repository=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
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

apps:
  - name: payroll
    secrets:
      USERNAME: opszero
    autoscaling:
      enabled: true
    service:
      enabled: true
      type: ClusterIP
      port: 3000
      hosts:
        - host: yieldpayroll.com
          paths: ["/"]
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
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
      port: 3000
      command: ["bundle", "exec", "rails", "server"]
      hosts:
        - host: yieldpayroll.com
          paths: ["/"]
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
