
# Manual Setup Steps

Terraform won't create the s3 bucket used for terraform remote state, you must create it by hand:

    aws s3 mb s3://jenkins201-tfremotestatenic


# Further Reading

* Container to run tflint in: https://github.com/dxw/testing-terraform-docker/blob/master/Dockerfile
* Based on https://github.com/nicholasjackson/terraform-cfgmgmtcamp
