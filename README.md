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


### \
Here's an updated example that appends the existing `assume_role_policy` to the new policy:

```hcl
module "my_iam_role" {
  source  = "./my_iam_role_module"
  name    = "my-iam-role"
  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
}

resource "aws_iam_policy_document" "updated_policy" {
  statement {
    effect    = "Allow"
    actions   = ["sts:AssumeRole"]
    principals {
      type        = "AWS"
      identifiers = [module.my_iam_role.role_arn]
    }
  }

  // Appending the existing assume_role_policy
  statement {
    effect    = "Allow"
    actions   = ["sts:AssumeRole"]
    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}

resource "aws_iam_role_policy" "example" {
  role   = module.my_iam_role.role_name
  policy = aws_iam_policy_document.updated_policy.json
}
```

In this example, the `aws_iam_policy_document` resource includes two `statement` blocks. The first block defines the new policy with the additional principal using the `module.my_iam_role.role_arn`. The second block appends the existing `assume_role_policy` from the `my_iam_role` module.

When creating the `aws_iam_role_policy`, the `policy` argument references the JSON representation of the `aws_iam_policy_document.updated_policy`.

With this approach, you combine the existing `assume_role_policy` from the `my_iam_role` module with the new policy using the `aws_iam_policy_document` resource, and then apply the updated policy to the IAM role using the `aws_iam_role_policy` resource.


```





### OPTION 3
To append a new statement to the existing trust policy of an IAM role, you'll need to use the `aws_iam_role_policy` resource with the `aws_iam_policy_document` data source. Here's an example of how you can append a new statement to the existing trust policy:

```hcl
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

In this example, we use two `aws_iam_policy_document` data sources: `existing_policy` and `updated_policy`. The `existing_policy` data source represents the existing trust policy of the IAM role, while the `updated_policy` data source represents the new statement you want to append to the existing policy.

In the `aws_iam_role_policy` resource, we use the `jsonencode` function to encode the policy document, similar to the previous example. However, this time, we use the `concat` function to concatenate the statements of the existing policy and the updated policy together.

By using `concat`, we combine the statements from both policy documents into a single list of statements, effectively appending the new statement to the existing trust policy.

Make sure to replace the values in the example with your own IAM role name and ARN as needed. When you apply this configuration, it will update the trust policy of the IAM role within the module by appending the new statement to the existing policy.




### OPTON 4
If you want to combine the two principals into a single statement, you can modify the code as follows:

```hcl
module "my_iam_role" {
  source = "./my_iam_role_module"
  name   = "my-iam-role"
}

data "aws_iam_policy_document" "combined_policy" {
  statement {
    effect = "Allow"
    actions = ["sts:AssumeRole"]

    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }

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
    Statement = data.aws_iam_policy_document.combined_policy.statement
  })
}
```

In this modified code, we create a new `aws_iam_policy_document` called `combined_policy` that contains a single statement. The statement includes both principals: the EC2 service principal and the specific AWS IAM role principal. The effect and actions remain the same.

The resulting policy document will have a single statement that includes both principals within the `principals` block.



### OPTION 4 FIX STATEMRNTS
Apologies for the confusion. The syntax error you're encountering is because the `data "aws_iam_policy_document" "combined_policy"` block does not support the `statement` block directly. Instead, you can use the `statements` argument to define multiple statements within the policy document.

Here's the corrected code:

```hcl
module "my_iam_role" {
  source = "./my_iam_role_module"
  name   = "my-iam-role"
}

data "aws_iam_policy_document" "combined_policy" {
  statement {
    effect = "Allow"
    actions = ["sts:AssumeRole"]

    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }

  statement {
    effect = "Allow"
    actions = ["sts:AssumeRole"]

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
    Statement = data.aws_iam_policy_document.combined_policy.statements
  })
}
```

In this updated code, the `data "aws_iam_policy_document" "combined_policy"` block includes two separate `statement` blocks to define two individual statements. The `statements` argument is used in the `aws_iam_role_policy` resource to refer to the list of statements within the combined policy document.

Please give this updated code a try, and it should resolve the syntax errors you were facing.




### OPTION 6
Apologies for the confusion in my previous response. If you want to create an IAM role with its own ARN as a principal in the assume role policy, you cannot use the same IAM role resource to create and reference itself in the policy. It creates a cyclic dependency, which is not allowed in Terraform.

To achieve this, you can create the IAM role without specifying the assume role policy initially, and then use a separate `aws_iam_role_policy` resource to attach the assume role policy with multiple statements, including the IAM role's own ARN. Here's an example:

```hcl
resource "aws_iam_role" "example_role" {
  name = "example-role"
}

resource "aws_iam_role_policy" "example_policy" {
  name   = "example-policy"
  role   = aws_iam_role.example_role.name
  policy = jsonencode({
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "ec2.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      },
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": aws_iam_role.example_role.arn
        },
        "Action": "sts:AssumeRole"
      }
    ]
  })
}
```

In this example, the `aws_iam_role` resource creates the IAM role named "example-role" without specifying the assume role policy. Then, the `aws_iam_role_policy` resource attaches the assume role policy to the IAM role using the `role` attribute and includes the IAM role's own ARN as a principal in the policy document.

By separating the IAM role creation and assume role policy attachment into two resources, you can avoid the cyclic dependency issue.
