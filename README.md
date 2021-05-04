# terraform-acton

A composite action that checks format, init, validate, plans, deploys and generate json output from terraform scripts. It expects you have done terraform setup and credential configuration

## Usage

| Input               | Require | Default          | Description                                                        |
| ------------------- | ------- | ---------------- | ------------------------------------------------------------------ |
| `apply`             | `false` | `false`          | should the infrastructure change be applied                        |
| `working-directory` | `false` | `infra`          | directory to run terraform in                                      |
| `backend-config`    | `false` | `backend.config` | path to backend config, if set `false` will not use backend config |

| Output | Description                            |
| ------ | -------------------------------------- |
| `json` | json output from `terraform out -json` |

### Just plan

```yaml
jobs:
  # your job and other config
  steps:
    # terraform setup and credential configuration steps go here
    - uses: yogeshlonkar/terraform-action
```

### Apply and output

```yaml
jobs:
  # your job and other config
  steps:
    # terraform setup and credential configuration steps go here
    - id: terraform
      uses: yogeshlonkar/terraform-action
      with:
        apply: true
    - run: echo ${{ fromJson(steps.terraform.outputs.json).output_name.value }}
```

### With terraform and AWS config

```yaml
name: Continuous Integration
on:
  push:
    branches-ignore:
      - main
jobs:
  tf-plan:
    runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v2
    - name: AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}
    - name: Terraform Setup
      uses: hashicorp/setup-terraform@v1
      with:
        terraform_version: 0.15.0
    - id: terraform
      uses: yogeshlonkar/terraform-action
      with:
        backend-config: false
        working-directory: terraform/
        apply: false
    - run: echo ${{ fromJson(steps.terraform.outputs.json).output_name.value }}
```
