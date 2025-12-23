# AWS Indirect Node Query Testing

This project tests the AAP indirect node counting functionality using AWS free-tier resources.

## Contents

```
aws-query-test/
├── collections/
│   └── requirements.yml    # Ansible collection dependencies
├── playbooks/
│   ├── generate_aws_queries.yml  # Generate event_query.yml for amazon.aws
│   └── test_aws_queries.yml      # Test with free-tier AWS resources
├── requirements.txt        # Python dependencies
└── README.md
```

## Prerequisites

### Python Dependencies
```bash
pip install -r requirements.txt
```

### Ansible Collections
```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Usage

### Local Execution

```bash
# Set AWS credentials
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export AWS_REGION="us-east-1"

# Generate event query file
ansible-playbook playbooks/generate_aws_queries.yml

# Test with free-tier resources
ansible-playbook playbooks/test_aws_queries.yml
```

### AAP Execution

#### Credential Type: Amazon Web Services

Create an AWS credential in AAP with:
- Access Key
- Secret Key
- Region (optional)

#### Extra Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `aws_access_key_id` | AWS Access Key ID | From credential |
| `aws_secret_access_key` | AWS Secret Access Key | From credential |
| `aws_security_token` | AWS Session Token (for assumed roles) | None |
| `aws_region` | AWS Region | `us-east-1` |
| `resource_prefix` | Prefix for test resources | `aap-indirect-test` |
| `cleanup` | Delete resources after test | `true` |

#### Example Extra Vars (AAP Job Template)

```yaml
aws_region: us-west-2
resource_prefix: my-test
cleanup: false
```

#### To Keep Resources for Inspection

```yaml
cleanup: false
```

## AWS Resources Created (Free Tier)

| Resource | Module | Countable |
|----------|--------|-----------|
| S3 Bucket | `s3_bucket` / `s3_bucket_info` | Yes |
| SNS Topic | `sns_topic` / `sns_topic_info` | Yes |
| SQS Queue | `sqs_queue` / `sqs_queue_info` | Yes |
| IAM Role | `iam_role` / `iam_role_info` | Yes |
| Security Group | `ec2_security_group` / `ec2_security_group_info` | Yes |

## Validating Indirect Node Counts

After running in AAP, check the audit table:

```sql
SELECT name, canonical_facts, events, count 
FROM main_indirectmanagednodeaudit 
WHERE events::text LIKE '%amazon.aws%';
```

Expected: Each AWS resource should appear with its ARN as the canonical fact.

## Required IAM Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation",
        "sns:CreateTopic",
        "sns:DeleteTopic",
        "sns:ListTopics",
        "sqs:CreateQueue",
        "sqs:DeleteQueue",
        "sqs:ListQueues",
        "sqs:GetQueueAttributes",
        "iam:CreateRole",
        "iam:DeleteRole",
        "iam:GetRole",
        "iam:ListRoles",
        "ec2:CreateSecurityGroup",
        "ec2:DeleteSecurityGroup",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeRegions",
        "sts:GetCallerIdentity"
      ],
      "Resource": "*"
    }
  ]
}
```

## License

Apache 2.0
