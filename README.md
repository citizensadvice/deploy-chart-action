# Deploy Chart Action

This action will deploy a Helm chart to a cluster with an easy to use interface.

## Inputs

| Input              | Description                                                                    | Required | Default             |
| ------------------ | ------------------------------------------------------------------------------ | -------- | ------------------- |
| `deployment_name`  | The name of the deployment, also used as the namespace name                    | Yes      |                     |
| `role_arn`         | The ARN of the role to assume                                                  | Yes      |                     |
| `chart_repository` | The name of the image repository. Will be used as the name of the Docker image | No       | `github.repository` |
| `chart_path`       | The path within the repository to the chart                                    | No       | `chart`             |
| `chart_pull_token` | A token to pull the chart repository                                           | Yes      |                     |
| `chart_pull_path`  | The path within the runner to pull to the chart into.                          | No       | `.`                 |
| `cluster`          | The cluster to deploy to                                                       | No       | `dev-eks-platform`  |
| `extra_helm_flags` | Extra flags to pass to helm                                                    | No       | ``                  |

## Example Usage

```yaml
name: Deploy Chart
on:
  push:
    branches:
      - main
    tags:
      - v*

jobs:
  deploy:
    name: Deploy chart
    runs-on: ubuntu-22.04
    steps:
      - name: Deploy chart
        uses: citizensadvice/deploy-chart-action@v2
        with:
            deployment_name: <deployment name here>
            role_arn: <role arn>
            chart_repository: citizensadvice/helm-charts
            chart_path: charts/<your chart>
            chart_pull_token: ${{ secrets.HELM_CHARTS_PULL_TOKEN }}
            extra_helm_args: -f overrides.yaml
```

## Caveats

- This action assumes that the namespace has already been created. The namespace should be the same as the `deployment_name` input variable. If there is demand for the action to also create a namespace/subnamespace, this can be added.
- The IAM role being used must have permission to deploy the required resources in the namespace of the same name as the release. The role should be created using the [OIDC project](https://github.com/citizensadvice/aws-oidc-cdk) and use the Helm release role template.
