# Getting Started with Terraform

HashiCorp Terraform is an infrastructure as code tool that allows you to define, provision, and manage infrastructure resources via human-readable configuration files. 

Terraform works with an ecosystem of [providers](https://developer.hashicorp.com/terraform/language/providers), each of which allow Terraform to interact with an external API to provision and manage resources. In this tutorial you will create and run a Terraform configuration file that uses the Terraform Docker provider to create an NGINX instance in Docker. 


## Prerequisites

To follow this tutorial, you will need: 

* The Terraform CLI (version 1.0 or later). Visit [Install Terraform](https://developer.hashicorp.com/terraform/downloads) to download the correct binary package for your operating system.

* Docker Desktop. Visit [Docker Desktop](https://www.docker.com/products/docker-desktop/) to install the correct package for your operating system. 


## Create your configuration file

With Terraform installed, you are ready to create your first configuration file. 

Create a new directory on your local machine to store this new configuration. 

```shell
$ mkdir terraform-demo
```

Change into that directory. 

```shell
$ cd terraform-demo
```

Create a file for your Terraform configuration code.

```shell
$ touch main.tf
```

Next, you will use [Terraform's configuration language](https://developer.hashicorp.com/terraform/language) to declare what infrastructure resources you want to create. 

Open `main.tf` in your text editor, paste in the following configuration, and save the file.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

You now have a complete configuration file that you can deploy with Terraform. 

## Deploy your resources

Start Docker Desktop so that Terraform can communicate with it to create your resources. 

```shell
$ open -a Docker
```

Initialize Terraform with the `init` command.

```shell
$ terraform init
```

Terraform will initialize the backend and install the [Docker provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs), as required in your configuration file. 


```shell
Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Provision the resources with the `apply` command.

```shell
$ terraform apply
```

`terraform apply` may take a few minutes to run. When the process completes, you will be asked to confirm that you want to create these resources.

``` shell
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "training"
      + network_data                                = (known after apply)
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "no"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + image_id    = (known after apply)
      + name        = "nginx:latest"
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
```

Type `yes` and hit `Enter` to confirm that you want to create these resources. 

```shell
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 9s [id=sha256:25d0f5e5d6da4d2239c0162571062e340c2d4b9b6b3190c07e4ca087c3929371nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 0s [id=a99b1a3ec0c07075b727d910ebe07efa2e41168bcf6f2312d18b5d43712da6cf]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

You can visit the Docker Desktop UI to view your new `training` NGINX image.

## Destroy your resources 

Run `terraform destroy` to destroy the infrastructure created in this tutorial. 

```shell
$ terraform destroy
```

```shell
docker_image.nginx: Refreshing state... [id=sha256:25d0f5e5d6da4d2239c0162571062e340c2d4b9b6b3190c07e4ca087c3929371nginx:latest]
docker_container.nginx: Refreshing state... [id=a99b1a3ec0c07075b727d910ebe07efa2e41168bcf6f2312d18b5d43712da6cf]

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach                                      = false -> null
      - command                                     = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - container_read_refresh_timeout_milliseconds = 15000 -> null
      - cpu_shares                                  = 0 -> null
      - dns                                         = [] -> null
      - dns_opts                                    = [] -> null
      - dns_search                                  = [] -> null
      - entrypoint                                  = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env                                         = [] -> null
      - group_add                                   = [] -> null
      - hostname                                    = "a99b1a3ec0c0" -> null
      - id                                          = "a99b1a3ec0c07075b727d910ebe07efa2e41168bcf6f2312d18b5d43712da6cf" -> null
      - image                                       = "sha256:25d0f5e5d6da4d2239c0162571062e340c2d4b9b6b3190c07e4ca087c3929371" -> null
      - init                                        = false -> null
      - ipc_mode                                    = "private" -> null
      - log_driver                                  = "json-file" -> null
      - log_opts                                    = {} -> null
      - logs                                        = false -> null
      - max_retry_count                             = 0 -> null
      - memory                                      = 0 -> null
      - memory_swap                                 = 0 -> null
      - must_run                                    = true -> null
      - name                                        = "training" -> null
      - network_data                                = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - mac_address               = "02:42:ac:11:00:02"
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode                                = "default" -> null
      - privileged                                  = false -> null
      - publish_all_ports                           = false -> null
      - read_only                                   = false -> null
      - remove_volumes                              = true -> null
      - restart                                     = "no" -> null
      - rm                                          = false -> null
      - runtime                                     = "runc" -> null
      - security_opts                               = [] -> null
      - shm_size                                    = 64 -> null
      - start                                       = true -> null
      - stdin_open                                  = false -> null
      - stop_signal                                 = "SIGQUIT" -> null
      - stop_timeout                                = 0 -> null
      - storage_opts                                = {} -> null
      - sysctls                                     = {} -> null
      - tmpfs                                       = {} -> null
      - tty                                         = false -> null
      - wait                                        = false -> null
      - wait_timeout                                = 60 -> null

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:25d0f5e5d6da4d2239c0162571062e340c2d4b9b6b3190c07e4ca087c3929371nginx:latest" -> null
      - image_id    = "sha256:25d0f5e5d6da4d2239c0162571062e340c2d4b9b6b3190c07e4ca087c3929371" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:f4e3b6489888647ce1834b601c6c06b9f8c03dee6e097e13ed3e28c01ea3ac8c" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

Type `yes` and hit `ENTER` to confirm that you want to destroy these resoruces. 

Terraform will now destroy the resources that were created by `terraform apply`.

```shell
docker_container.nginx: Destroying... [id=a99b1a3ec0c07075b727d910ebe07efa2e41168bcf6f2312d18b5d43712da6cf]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:25d0f5e5d6da4d2239c0162571062e340c2d4b9b6b3190c07e4ca087c3929371nginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

## Next steps 

In this tutorial you created your first infrastructure resource using a Terraform configuration file. Continue to the [next tutorial](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-change) to learn how to modify your infrastructure with Terraform.   

To learn more about fundamental Terraform concepts and practical use cases, check out these links from the HashiCorp Developer center: 

* [What is Terraform?](https://developer.hashicorp.com/terraform/intro)
* [The Core Terraform Workflow](https://developer.hashicorp.com/terraform/intro/core-workflow)
* [Use Cases](https://developer.hashicorp.com/terraform/intro/use-cases)