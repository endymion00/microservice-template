### Microservice Template ([see video](foo))

The scalable monitored template for cluster infrastructure.

* Provision servers with Packer and Chef
* Deploy and scale with Terraform
* Service discovery with Consul
* Data collection with collectd and statsd
* Analysis and visualization with InfluxDB and Grafana

#### Installation

1. Sign up for Amazon Web Services free tier [here](http://aws.amazon.com/free/)
    **Extremely important**: ensure the region in your Amazon Web
    Console is set to "N. California" or future steps will fail
    mysteriously.
2. Create Amazon IAM credentials for deployment.
    Assuming you are signed into the AWS console, visit the [user
    admin](https://console.aws.amazon.com/iam/home#users) page. Create
    a new user and note their security credentials (ID and secret key).
    Then attach a user policy of "Power User" to the newly created user.

3. Clone this repo including its chef recipe submodules.
    ```bash
    git clone --recursive https://github.com/begriffs/microservice-template.git

    # if you've already cloned the repo you can do: git submodule update --init
    ```

4. Install <a href="https://www.packer.io/" target="_blank">Packer</a> and
    <a href="https://www.terraform.io/" target="_blank">Terraform</a>. On a
    Mac you can install them with homebrew:
    ```bash
    brew tap homebrew/binary
    brew install packer
    brew install terraform
    ```

5. Create machine images (AMI) using your credentials.
    When running each of these commands **write down** the AMI ids
    each one generates. They will be of the form `ami-[hash]`. You
    will need to remember which command created which AMI.

    ```bash
    packer build -var 'aws_access_key=xxx' -var 'aws_secret_key=xxx' consul.json
    packer build -var 'aws_access_key=xxx' -var 'aws_secret_key=xxx' statsd.json
    packer build -var 'aws_access_key=xxx' -var 'aws_secret_key=xxx' influx.json
    packer build -var 'aws_access_key=xxx' -var 'aws_secret_key=xxx' grafana.json
    ```

6. Deploy machine images.
    Edit `terraform/vars.tf` and fill in the ami instances created by
    the previous steps. Leave the `aws` keys blank.

    Copy the Terraform variables example to a file without the `.example`
    and fill it in.
    ```bash
    cp terraform/terraform.tfvars{.example,}
    ```

    Now the fun part. Go into the terraform directory and run `make`.

    At the end it will output the public IP address of the monitoring
    server for the cluster. You can use it to watch server health and
    resource usage.

#### Monitoring

The server exposes web interfaces for several services.

Port | Service
---- | -------------------------------------
80   | [Grafana](http://grafana.org/) charts
8500 | Consul server status and key/vals
8080 | InfluxDB admin panel
8086 | InfluxDB API used by Grafana

Influx has been configured with two databases, `metrics` and
`grafana`. Cluster data accumulates in the former and Grafana stores
your chart settings in the latter. The Influx user `grafana` (password
`grafpass`) has full access to both tables.

#### Collecting more stats

The cluster exposes [StatsD](https://github.com/etsy/statsd/)
server(s) at `statsd.node.consul`. Your applications should send
it lots of events. The statsd protocol is UDP and incurs little
application delay. The statsd server relays all info to InfluxDB
which makes it accessible for graphing.