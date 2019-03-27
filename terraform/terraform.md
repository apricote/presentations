<img src="./terraform_logo.svg" style="padding: 20px" alt="terraform logo">

## Theory and Practice

---

### Content

- About me
- Infrastructure as Code
- What is Terraform?
- <span style="color: #e74c3c">narando</span> Backstory
- Terraform at <span style="color: #e74c3c">narando</span>
- What is planned?
- Recap

---

### About me

![picture of me](https://www.gravatar.com/avatar/b1d145930b4db0d59a3d40df3688340f.jpg?s=200)

Julian Tölle  
Developer @ <span style="color: #e74c3c">narando</span> & <span style="color: #f2f2f2">TrackCode</span>  
Backend Development & DevOps

--

### About us

![narando Logo](./narando_logo.png) <!-- .element style="width: 50%" -->

- ~ 5 developers
- crowd-sourced audio production for blogs
- Node.js on AWS

---

### Infrastructure as Code

#### Domain

- Provisioning servers
- Configuring databases
- Firewall rules
- DNS
- configuring and deploying applications

--

### Infrastructure as Code

#### Before IaC

- servers and applications are configured manually
- new environments take days/weeks to be running
- copy&paste
- search&replace

--

### Infrastructure as Code

#### Definition

> Infrastructure as code (IaC) is the process of managing [...] data centers through machine-readable definition files, rather than [...] interactive configuration tools.

<small>From [Wikipedia](https://en.wikipedia.org/wiki/Infrastructure_as_code)</small>

--

### Infrastructure as Code

#### Benefits

- automation saves (expensive) time
- avoids human errors
- easier to review/verify

--

### Infrastructure as Code

#### Tools

- Chef
- Puppet
- Ansible
- AWS CloudFormation
- Terraform

---

### What is Terraform?

- open-source toolchain for IaC
- declarative approach
- providers for every big cloud

--

### What is Terraform?

#### HCL

```go
provider "aws" {
  access_key = "ACCESS_KEY_HERE"
  secret_key = "SECRET_KEY_HERE"
  region     = "eu-central-1"
}

resource "aws_instance" "www" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

--

### What is Terraform?

#### Resource Graph

```go
resource "aws_instance" "www" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}

resource "cloudflare_record" "www" {
  domain = "example.com"
  type   = "A"
  value  = "${aws_instance.www.public_ip}"
}
```

--

### What is Terraform?

#### Modules

- `*.tf` in a folder
- share a namespace
- input is called `variable`
- output is called `output`

```go
variable "env" {
  type    = "string"
  default = "prod"
}

output "public_ip" {
  type  = "string"
  value = "${aws_instance.www.public_ip"
}
```

--

### What is Terraform?

##### State

- contains all managed resources - "why does the container not receive traffic?"
  - might be AWS specific
-

* used to produce "execution plan"
* JSON format
* can be local or remote (S3, TF Enterprise, etc.)

--

### What is Terraform?

#### `terraform plan`

1. Verify variables
2. Load state
3. Refresh resources
4. Diff state against desired state
5. Plan actions to reach desired state

--

### What is Terraform?

#### `terraform apply`

1. Load generated plan
2. Apply actions in sequence

---

### <span style="color: #e74c3c">narando</span> Backstory

#### Development History

- **2014-2015**: initial app
- **2016**: Maintenance Mode
- **January 2017**: Rewrite
- **~ May 2017**: live traffic for new services

--

### <span style="color: #e74c3c">narando</span> Backstory

#### Initial App

- Built by previous co-founder
- Ruby on Rails
- Heroku
  - Addons for DB, Cache, DNS, Logs, Monitoring
- Only **prod** environment

> "Let's test this in production!"

--

### <span style="color: #e74c3c">narando</span> Backstory

#### Rewrite

- **Requirement**: multiple environments
- Node.js services
- Docker + Elastic Container Service
- All resources in AWS

---

### Terraform at <span style="color: #e74c3c">narando</span>

#### Initial Struggles

- AWS resources managed by hand
- scripts for deployments
- manually duplicating setup for **prod**

-

* new system went live in May 2017
* modifications became dangerous

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Rev. I - Proof of Concept

- one file with all resources
  - ECS, EC2, VPC, RDS, Elasticache, IAM, ALB, Route53
  - each with 1~5 TF resources
- only prod environment
- 554 LOC

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Rev. II - Adding Structure

- `modules/`
  - encapsulates concern
  - "central" modules like `modules/vpc`, `modules/cluster`, `modules/iam`
  - "once-per-service" - `module/service`
    - Container definition
    - Loadbalancer Target
    - DNS Entry
    - Firewall Rules

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Rev. II - Adding Structure

- `env/$ENV`
  - includes all modules
  - root for state
  - difference between environments
    - **prod**: Elasticache
    - **dev**: custom Redis
    - same inputs/outputs
    - can be easily swapped

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Rev. II - Directory Tree

```
├── env
│   ├── dev
│   │   ├── main.tf
│   │   ├── services.tf
│   │   └── vars.tf
│   └── prod
│       └── same as dev
└── modules
    ├── cluster
    │   ├── vars.tf
    │   ├── outputs.tf
    │   └── main.tf
    ├── service
    ├── dns
    └── vpc
```

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Rev. II - services.tf

- one `module/service` per service
- similar services
  - db
  - cache
  - hostname
  - container definition
- different services in **dev**/**prod**

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Rev. III - Splitting of services.tf

- services became more custom, more resources
- moved into own module
- 1 file per service

```
└── env
    └── dev (prod)
        └── services
            ├── feed-fetcher.tf
            ├── feeds.tf
            ├── notifier.tf
            ├── narrator.tf
            ├── publisher.tf
            ├── vars.tf
            └── outputs.tf
```

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Pain Points (Structure)

- state updates are slow
- infrastructure is seperate from code (+CI/CD)
- not all values are auto-generated(/-filled)
- service definitions duplicated

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Pain Points (Process)

- manual `apply` by me
  - repo might differ from deployed infrastructure
- CI does not create plans to review
  - reviewers need to do this themselves
- iteration is slow
  - missing `playground` environment
  - smoke tests?

--

### Terraform at <span style="color: #e74c3c">narando</span>

#### Pain Points (Terraform)

- not all service providers have terraform providers
  - drone / CICD
  - gitea / VCS
  - pointDNS
- some validations are only done during `apply`

---

### What is planned?

#### Upgrade Modules to support cross-repo setups

- Using `data` resources and naming schemas
- allows moving service-specific code into repos

```go
data "aws_ecs_cluster" "cluster" {
  cluster_name = "${var.env}"
}
```

--

### What is planned?

#### Move service definitions into service repos

- integrated with usual review and deployment processes
- improves state update time
- single PR for related changes
  - e.g. implement a new batch task (service repo)
  - and configure the cron trigger (currently infrastructure repo)

--

### What is planned?

#### Automatic apply

- implement CD for core infrastructure repo
- probably with manual review at first

--

### What is planned?

#### Move Root DNS

- to Service Provider with Terraform Provider
  - AWS / OVH / Cloudflare
- less manual work

---

### Recap

#### Things I like

- Exhaustive documentation
- huge list of offical providers
- community-built providers for new products
- declarative approach like K8s
- minial code, all configuration
- Hcl looks like Go and feels like Go

--

### Recap

#### Things I do not like

- side-effects between resources are not always documented
  - searching in the dark for errors
  - "why does the container not receive traffic?"
  - might be AWS specific
- lots of concepts to learn
- failures do not roll back
  - changes must be self-contained and backwards compatible
