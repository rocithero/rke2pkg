# rke2pkg


```
module "my_iam_role" {
  source = "./my_iam_role_module"
  name   = "my-iam-role"
}

data "aws_iam_policy_document" "existing_policy" {
  statement {
    effect    = "Allow"
    actions   = ["sts:AssumeRole"]
    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}

data "aws_iam_policy_document" "updated_policy" {
  statement {
    effect    = "Allow"
    actions   = ["sts:AssumeRole"]
    principals {
      type        = "AWS"
      identifiers = ["arn:aws:iam::123456789012:role/other-role"]
    }
  }
}

resource "aws_iam_role_policy" "example" {
  role   = module.my_iam_role.role_name

  policy = jsonencode({
    Version   = "2012-10-17"
    Statement = concat(
      data.aws_iam_policy_document.existing_policy.statement,
      data.aws_iam_policy_document.updated_policy.statement
    )
  })
}

```
