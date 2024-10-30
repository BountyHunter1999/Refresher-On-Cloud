# Refresher on Cloud Tools

## LocalStack

- https://docs.localstack.cloud/get-started/
- run everything with `docker compose up`
  - `--endpoint-url=http://localhost:4566` to talk to localstack when using the normal CLI
  - else just use `awslocal`

### S3

- `awslocal s3 mb s3://my-bucket`
    - `aws s3 mb s3://my-bucket --endpoint-url=http://localhost:4566`
- `awslocal s3 ls`
    - when using normal CLI use the endpoint
    - `aws s3 ls --endpoint-url=http://localhost:4566`