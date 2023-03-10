# Deploy Chart Action

This action will deploy a Helm chart to a cluster with an easy to use interface.

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `deployment_name` | The name of the deployment, also used as the namespace name | Yes | |
| `aws_access_key` | AWS access key ID | Yes | |
| `aws_secret_key` | AWS secret access key | Yes | |
| `chart_repository` | The name of the image repository. Will be used as the name of the Docker image | No | `github.repository` |
| `chart_path` | The path within the repository to the chart | No | `chart` |
| `pull_token` | A token to pull the chart repository | No | `GITHUB_TOKEN` |
| `cluster` | The cluster to deploy to | No | `devops-eks-controlplane` |
| `helm_values` | A newline seperated list of helm values, excluding the `--set` flag | No | |
| `helm_values_file` | Path to a helm values file | No | |

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
      - name: Build and push to ECR
        uses: citizensadvice/build-and-push-private-action@v1
        with:
            aws_access_key: ${{ secrets.PUBLIC_PUSH_ECR_AWS_KEY }}
            aws_secret_key: ${{ secrets.PUBLIC_PUSH_ECR_AWS_SECRET }}
            dockerfile_context: '.'
            repository_name: <REPOSITORY NAME HERE>
            auth_token: ${{ secrets.GITHUB_TOKEN }}

      - name: Deploy chart
        uses: citizensadvice/deploy-chart-action@v1
        with:
            deployment_name: <deployment name here>
            aws_access_key: ${{ secrets.AWS_ACCESS_KEY }}
            aws_secret_key: ${{ secrets.AWS_SECRET_KEY }}
            chart_repository: citizensadvice/helm-charts
            chart_path: charts/<your chart>
            pull_token: ${{ secrets.HELM_CHARTS_PULL_TOKEN }}
            helm_values: |
                image.tag=1.2.3
                foo=bar
            helm_values_file: .github/resources/overrides.yaml
```

## Caveats

- This action assumes that the namespace has already been created. The namespace should be the same as the `deployment_name` input variable. If there is demand for the action to also create a namespace/subnamespace, this can be added.
- The IAM user being used must have permission to deploy the required resources in the namespace of the same name as the release
