# Manual-Code-Reviews

Best Practices for Ensuring Code Quality, Security, and Maintainability

## Summary

Manual code reviews are essential because they help identify and fix issues that automated tools might miss, such as logic errors, design flaws, and subtle security vulnerabilities. They also foster collaboration and knowledge sharing among team members, leading to improved code quality and adherence to best practices. Additionally, manual reviews promote accountability and a deeper understanding of the codebase, contributing to better maintainability and long-term project success.

## Guidelines

### Manual Code Reviews

Code Readability - Easy to read, clear function names, variables, and concise comments to articulate what’s taking place.

Code Structure - Well organized code and modular for expedited troubleshooting.

Error Handling - Ensure actions such as try-catch blocks are used and have meaningful error messages for debugging and end user support.

Below is an example of doing this in Terraform, if there are approved ec2 instance types:

```
variable "instance_type" {
  description = "EC2 instance type"
  type        = string

  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "Instance type must be one of 't2.micro', 't2.small', 't2.medium'."
  }
}
```
### Security Specific

Input Validation - Any input is canonical and formatted to meet the expected requirements of the process that is running.

In situations where variables or other pieces of information are being pulled into the function, the following can be used to validate if the variable is empty or not:

```
if [[ -z "$EXPECTED_ENV_VAR" ]]; then
  echo "Error: EXPECTED_ENV_VAR is not set"
  exit 1
fi
```

Here is another example on how to handle this:

```
variable "instance_type" {
  description = "EC2 instance type"
  type        = string

  validation {
    condition     = contains(["t2.micro", "t2.small", "t2.medium"], var.instance_type)
    error_message = "Instance type must be one of 't2.micro', 't2.small', 't2.medium'."
  }
}

variable "instance_count" {
  description = "Number of instances"
  type        = number

  validation {
    condition     = var.instance_count > 0 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}
```

Authentication/ Authorization - If any sort of authentication/ authorization takes place, ensure that hard coded credentials aren't being used to provide the process with permissions, and in addition to this, any authen/ author blocks have if/ else statements that validate whether the credentials are valid.

Below are some examples on how to do this in Terraform, Python, and Bash:

#### Terraform

```
variable "my_variable" {
  type = string
}

resource "null_resource" "example" {
  count = var.my_variable != "" ? 1 : 0

  provisioner "local-exec" {
    command = "echo Variable is set to ${var.my_variable}"
  }
}

output "error_message" {
  value = "Error: my_variable is not set"
  count = var.my_variable == "" ? 1 : 0
}
```

#### Python

```
value = "non_empty_value"
result = 1 if value != "" else 0
print(result)  # This will print 1
```

#### Bash

```
value="non_empty_value"
if [[ -n "$value" ]]; then
  result=1
else
  result=0
fi
echo $result  # This will print 1
```

Keep in mind, these are all basic methods to checking for validation (whether or not the value exist). These should be modified to address the specific concern or issue you want to validate for. Clear and consice error messages are also ideal to use in these situations to allow for effective debudding or end user support. i.e. Your username or password are incorrect. 

Sensitive data - Look for any inefficient algorithms or code that could be optimized for better performance. i.e. ensure any algorithms that we are using are nist 800-53 approved for use based on the low, moderate, and high levels of data classification. This is simply used for this example, however, whatever compliance requirements the organization has should be followed.

Dependency Management - Review the use of all external libraries being called, and ensure that it is in line with our internal security standards. i.e. check the library being used and ensure there aren't any glaring vulnerabilities open, and if it impacts the specific function being used. Also, ensure you are only importing the function that you are going to use as opposed to the full library.

Resource Management - Define resource requests and limits for CPU and memory to ensure efficient resource utilization and prevent resource contention in Kubernetes deployments.

Namespace Management - Use namespaces to logically isolate resources and manage resource quotas at the namespace level.

Secrets - Manage application configurations and sensitive data securely using Secrets. Ensure that these Secrets are stored in encrypted formats and are masked to avoid being sent in plaintext during deployments or logging processes. Implement access controls to restrict access to Secrets only to authorized users and services.

Helm Chart Best Practices - Follow Helm chart best practices, such as maintaining chart versioning, using appropriate chart structure, and validating templates with helm lint.