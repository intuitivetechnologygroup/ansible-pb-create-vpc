ansible-pb-vpc
==============================

ansible playbook to create Cloudformation VPC Stacks given
a template of subnets to create.

## Components
<dl>
<dt>`vpc.yml` playbook</dt>
<dd>Generic entry point for VPC creation that also discovers the appropriate IntuitiveTech Nat AMI ( see below )
</dl>

## Nat

Currently configured to use generic Ubuntu AMIs for Nat.

A future consideration would be to switch over to Managed Nat service new to AWS VPC but it's not supported in CloudFormation so it would be an out-of-stack lifecyle mgmt task.

### Nat AMI Discovery

Since we aren't currently cutting generic AMIs, we have no IntuitiveTech AMIs to use for this task.

Instead of hard coding canonical source AMI ids into this repo, follow this process to create AMIs and they'll be available for us until we remove them ( unlike the upstream AMIs, which could disappear behind the ASGs whenever ).
1. Choose a recent source AMI Ubuntu 14.04 ( or probably later will work )
1. Copy the AMI to the same region
1. Set the name of the ami to the discovered name ( "intuitivetech-vpc-nat" )
1. You can leave the description as is, it clearly defines lineage
1. Once the AMI is available, the next invocation of the stack will update the NAT ASGs to use the new AMI.

Discovery is handled by a quick library module which simply invokes the boto3 describe_images call against the specified region.  It uses the Owners setting to specify 'self' so 3rd parties can't inject malicious AMIs in to our environment ( this also means we'll need a separate AMI for each region, for each account, but we could potentially use one account for these shared resources ), and additionally filters on the specified name.  It sorts the restults by creation date ( which is also a lexographically ordered string )

### Prerequisites:

Though moving to Credstash should make it technically possible to script, the automation currently _doesn't_ create:

* NAT SSH Key pair for the nat boxes.  By design this is named `{{ecosystem}}-{{environment}}-nat`
* AMI Image for the Nat boxes in each account/region.  You don't need a different one for each environment or VPC, just region.

### Execution

`ansible-playbook -vvvv -i inventory/ deploy.yml -e 'environ=dev region=us-west-2 ecosystem=intuitivetech'`

### Update Life Cycle

The stack process should be able to include most ongoing updates to the stacks, just be careful.  If in doubt, it would be best to create a new VPC from an old version, then update to a new version, to test, rather than testing directly into the dev VPC.  From the VPC mgmt point of view, dev is a production user!  That having been said, it's OK to iterate in dev, as long as lots of care is taken not to make a mess of things.
