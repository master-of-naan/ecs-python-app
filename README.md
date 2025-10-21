# ECS Python Flask Application

A simple Python Flask application deployed on AWS ECS Fargate with complete infrastructure as code and CI/CD pipeline.

## Architecture

- **Application**: Python Flask (Hello World API)
- **Container Registry**: AWS ECR
- **Compute**: AWS ECS Fargate
- **Load Balancer**: Application Load Balancer (ALB)
- **Networking**: VPC with public subnets across 2 AZs
- **CI/CD**: GitHub Actions

## Project Structure

```
.
├── src/
│   ├── app.py              # Flask application
│   └── requirements.txt    # Python dependencies
├── infra/
│   ├── main.tf            # Terraform infrastructure
│   ├── variables.tf       # Terraform variables
│   └── outputs.tf         # Terraform outputs
├── .github/
│   └── workflows/
│       ├── deploy-infra.yml  # Infrastructure deployment
│       └── deploy-app.yml    # Application deployment
├── Dockerfile             # Container image definition
└── README.md
```

## Prerequisites

- AWS Account with appropriate permissions
- GitHub Account
- Terraform >= 1.5.0
- Docker
- AWS CLI

## Local Development

### Run the application locally

```bash
cd src
pip install -r requirements.txt
python app.py
```

Access the app at `http://localhost:8080`

### Build and test Docker image

```bash
docker build -t ecs-python-app .
docker run -p 8080:8080 ecs-python-app
```

## Deployment

### 1. Setup GitHub Secrets

Add the following secrets to your GitHub repository:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

### 2. Deploy Infrastructure

The infrastructure will be automatically deployed when changes are pushed to the `infra/` directory.

Or manually deploy:

```bash
cd infra
terraform init
terraform plan
terraform apply
```

### 3. Deploy Application

The application will be automatically deployed when changes are pushed to `src/` or `Dockerfile`.

Or manually deploy:

```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

# Build and push
docker build --platform linux/amd64 -t <ecr-repo-url>:latest .
docker push <ecr-repo-url>:latest

# Update ECS service
aws ecs update-service --cluster ecs-python-app-cluster --service ecs-python-app-service --force-new-deployment
```

## API Endpoints

- `GET /` - Returns hello message with hostname and environment
- `GET /health` - Health check endpoint

## Infrastructure Components

- **VPC**: 10.0.0.0/16 with 2 public subnets
- **ALB**: Internet-facing load balancer
- **ECS Cluster**: Fargate cluster with Container Insights enabled
- **ECS Service**: 2 tasks running across 2 AZs
- **ECR**: Container image repository
- **CloudWatch**: Log aggregation
- **IAM**: Task execution and task roles

## Monitoring

CloudWatch Logs are available at: `/ecs/ecs-python-app`

## Cleanup

To destroy all resources:

```bash
cd infra
terraform destroy
```

## Security Notes

- Application runs as root user (for simplicity as requested)
- Single-stage Dockerfile (for simplicity as requested)
- ALB accepts HTTP traffic on port 80
- ECS tasks run in public subnets with public IPs

## Cost Estimation

Approximate monthly costs (us-east-1):
- ECS Fargate (2 tasks, 0.25 vCPU, 0.5 GB): ~$15
- ALB: ~$20
- Data transfer: Variable
- CloudWatch Logs: ~$1

**Total: ~$36/month** (excluding data transfer)
