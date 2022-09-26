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

redis:
  enabled: true # Enable Redis

apps:
  - name: payroll
    service:
      enabled: true
      type: ClusterIP
      port: 3000
      hosts:
        - host: yieldpayroll.com
          paths: ["/"]
    jobs:
      - name: db-migrate
        command: "bundle exec rake db:migrate"
    cronjobs:
      - name: hn
        command: ["bundle", "exec", "rails", "pay_people"]
        schedule: "0 * * * *"
  - name: payroll
    image: foobar:latest
    service:
      enabled: true
      type: ClusterIP
      port: 3000
      hosts:
        - host: yieldpayroll.com
          paths: ["/"]
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
```
