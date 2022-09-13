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
```