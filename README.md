> ## ⚠️ DEPRECATED — no longer used (PLA-75)
>
> This action has **zero consumers** across the TickX organisation and is being retired.
>
> It targets the long-dead `node12` runtime, and its `core.exportVariable` call relies on the
> legacy `::set-env::` workflow command — which is why consuming workflows had to set
> `ACTIONS_ALLOW_UNSECURE_COMMANDS: true`, disabling GitHub's protection against
> workflow-command injection.
>
> **Do not use this in new workflows.** Write the dotenv file with a native shell step instead:
>
> ```yaml
> - name: Write .env
>   env:
>     MY_KEY: ${{ secrets.MY_SECRET }}
>   run: echo "MY_KEY=$MY_KEY" >> .env
> ```
>
> Pass values through a step-level `env:` block rather than interpolating `${{ }}` directly into
> the `run:` body. GitHub automatically redacts `secrets.*` values from logs, so no explicit
> masking is needed.
>
> For secrets sourced from AWS at deploy time, prefer injecting them at runtime from SSM
> Parameter Store or Secrets Manager (e.g. an ECS task-definition `secrets` block) rather than
> baking them into a file or image layer.

![](https://github.com/TickX/aws-secret-to-dotenv/workflows/Test/badge.svg)
![GitHub license](https://img.shields.io/github/license/TickX/aws-secret-to-dotenv)

# AWS Secret to Dotenv

A GitHub action that appends an AWS Secret value(s) to a dotenv file.

## Usage

```yaml
steps:
  - uses: TickX/aws-secret-to-dotenv@v1.0.0
    with:
      secret: 'service-tickx/production' # [Required] This is the AWS secret name
      key: 'DB_URI' # [Optional] You can specify a specific key to fetch from the specified secret
      as: 'DB' # [Optional] You can provide an alternate name for the value retrieved using the specified `key`
      envPath: '.env' # [Optional] The path to the dotenv file (defaults to `.env`)
```

## Requirements

The following environment variables must be set as the corresponding IAM user credentials:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_DEFAULT_REGION`

The user must also have the following policy assigned to them:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "*"
        }
    ]
}
```
