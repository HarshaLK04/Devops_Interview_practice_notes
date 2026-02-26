**Terraform drift **
Terraform Drift happens when you create infra with Terraform but some manual changes added later which are not present in statefile this is called Terraform Drift. 
so to resolve this issue we use below steps:
- First check Terraform Plan it will highlight the drift
Or You can use 
driftctl scan
- if you want to keep changes and update in state file then use "update" command
E.X.
update instance_type in .tf file to t2.small
- if you don't want to do any changes then reapply use
Terraform apply

This question was asked to me in recent HCL interview
Note: this is also considered as answer for statefile corruption and apart from drift there is another solution also i.e. terraformer command I don't know much about it so unable to explain
