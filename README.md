# project001
This Repo is meant to serve as an example project repo that services 3 environments in Terraform Enterprise, specifically 3 unique worspaces that each map to a sub directory found in this repo. 

## Getting Started
This repo should run out of the box when correctly cloned and mapped to a TFE organization that you control.  Each sub directory has a maint.tf and variables.tf file that executes the provisioning of a single ec2 instance to a pre-created VPC and subnet that maps to each workspace Research, Test, & Prod.  Feel free to modify the main.tf, at a minimum update the tags on the instance.  The original demo was performed on a worksapce that had a basic Sentinel policy to check for tags on ec2 instances.  

## Example Sentinel policy
To demonstrate a basic Sentinel check for tags create a Sentinel policy in your TFE org:  

Policy Name: Check for Tags
Enforcement Mode: hard-mandatory (or your choice)  

```import "tfplan"

# Warning, this is case sensitive.  This is on purpose especially for organizations that do cost analysis on tag names.   Case sensitivity enforcements allow you to not have duplicates by case

mandatory_tags = [
  "TTL",
  "owner",
]

# Get all AWS instances contained in all modules being used
get_aws_instances = func() {
    instances = []
    for tfplan.module_paths as path {
        instances += values(tfplan.module(path).resources.aws_instance) else []
    }
    return instances
}

aws_instances = get_aws_instances()

# Instance tag rule
instance_tags = rule {
    all aws_instances as _, instances {
    	all instances as index, r {
            all mandatory_tags as t {
                r.applied.tags contains t
            }
        }
    }
}

# print("tfplan: %s", tfplan)
main = rule {
    (instance_tags) else true
}
```
