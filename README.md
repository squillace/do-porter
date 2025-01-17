# Spring Music on Kubernetes with Digital Ocean PostgreSQL

This example shows how you can use `Porter` to create a CNAB bundle that will use `s3cmd` to first make a new Space on Digital Ocean, use `Terraform` to provision a PostgreSQL Database Cluster, and deploy [Spring Music]() to a Kubernetes cluster with `Helm`. When `Terraform` is run, it will use the Space created by `s3cmd` as a backend, in order to store the TF state file. When `Helm` is used to deploy Spring Music, it will use the database connection information generated by the `Terraform` apply! This example shows how multiple tools can be chained together to deploy a Cloud Native Application.

This repo also contains an example GitHub action configuration `.github/workflows/main.yml` that shows how you might use `Porter` to build and publish the CNAB bundle to a Docker Registry!

## Try it Out Yourself

### Prerequisites

In order to try this out yourself, you'll first need to install `Porter`. Please see the [installation](https://porter.sh/install/) instructions for the steps needed to do that. You'll need to have [Docker installed](https://docs.docker.com/install/) in order to use `Porter`, and you'll also need a Digital Ocean [account](https://cloud.digitalocean.com/registrations/new).

Once you have the account, you'll need to create an [access token](https://www.digitalocean.com/docs/api/create-personal-access-token/) and generate a [Spaces API Key](https://www.digitalocean.com/community/tutorials/how-to-create-a-digitalocean-space-and-api-key).

You should set these as environment variables for use with the bundle. These values will look something like:

```
DO_SPACES_SECRET=bqPMB6k00MXkhOKucafebabea4LYphcnNCfOBL/+4tU
DO_SPACES_KEY=KIAGKdecafbadS5SSCRC
DO_ACCESS_KEY=16d225ebdecafbadcff2d05484c78ea8f81f38224940f3c4844c0cdd9996c75c
```

NOTE: You need the "secret" values for the Access Token and Spaces Secret, not the name.


Next, you'll need a Kubernetes cluster and a `kubeconfig` file. You can use [AKS](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal), [Digital Ocean](https://www.digitalocean.com/products/kubernetes/), any other Kubernetes cluster, as long as it can successfully create a service of type LoadBalancer.

### Clone This Repo

The first thing you'll need to do in order to try the bundle out is to clone this repo and enter into that directory:

```
git clone https://github.com/jeremyrickard/do-porter.git
cd do-porter
```

### Build The Bundle

Now, you are ready to build the bundle.

```
porter build
```

### Generate a Credential

Once this has finished, you are almost ready to install the bundle. Before you install, however, you'll need to generate a credential. To do this, we'll use the `porter credentials generate` command:

```
porter credentials generate
```

This will launch a guided dialog for building a credential. You'll see prompts like this:

```
Generating new credential spring-music from bundle spring-music
==> 4 credentials required for bundle spring-music
? How would you like to set credential "do_access_token"  [Use arrows to move, space to select, type to filter]
  specific value
> environment variable
  file path
  shell command
```

For the three digital ocean credentials, select `environment variable` and provide the corresponding value. For the `kubeconfig`, you should select `file path` and give the location of your config file:

```
? How would you like to set credential "kubeconfig"  [Use arrows to move, space to select, type to filter]
  specific value
  environment variable
> file path
  shell command
Enter the path that will be used to set credential "kubeconfig" $HOME/.kube/config
```

### Install The Bundle

Now that you have build the bundle and generated a credential, you are ready to install the bundle. To do that, we'll use the `porter install` command. You'll need to pick a value for the `space_name` and `database_name` parameter.


```
porter install -c spring-music --param space_name=<SOME VALUE> --param database_name=<SOME VALUE>
```

Some other parameters that you can set are:

* region
* node_count
* namespace
* helm_release


### Viewing the Output

Once the bundle has been successfully installed, you can use the `porter instances list` and `porter instances show` command to see your bundle installations.

First, view the instances that have been installed:

```
$ porter instances list
NAME           CREATED       MODIFIED      LAST ACTION   LAST STATUS
spring-music   4 hours ago   3 hours ago   install       success
```

Then, use `porter instances show` to view more details:

```
$ porter instances show spring-music
Name: spring-music
Created: 4 hours ago
Modified: 3 hours ago
Last Action: install
Last Status: success

Outputs:
-----------------------------------------------
  Name        Type    Value (Path if sensitive)
-----------------------------------------------
  service_ip  string  13.68.174.214
```

NOTE: the value for `service_ip` will be different for your installation

You should be able then view the application by navigating to that IP address in a web browser.
