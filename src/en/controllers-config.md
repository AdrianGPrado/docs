Title: General configuration options
TODO: Check accuracy of key table
      Make the table more space-efficient. Damn it's bulbous.
      error: controller-config shows config for a controller, not a model
      error: table's default value keys do not show up with above command (e.g. all bootstrap-*)
      error: --config configures the 'default' and 'controller' models
      "dynamically set by Juju" could use some explaination


# Configuring controllers

A Juju controller is the management node of a Juju cloud environment. It houses
the database and keeps track of all the models in that environment.

Controller configuration consists of a collection of keys and their respective
values. An explanation of how to both view and set these key:value pairs is
provided below.

## Listing configuration

A controller's configuration can be listed by running this command:

```bash
juju controller-config
```

The key-value pairs that are shown will include those that were set during
controller creation (see below), inherited as a default value (see table), or
dynamically set by Juju. 

## Setting configuration

To set a key's value use the `--config` option during the controller creation
process (see [Creating a controller][controllers-creating]). For example, to
create and configure a cloud named 'lxd':

```bash
juju bootstrap --config bootstrap-timeout=500 lxd
```

Once a controller is created, all its settings become immutable.

Note that the `--config` option may also be used to configure the 'controller'
and 'default' **models**. Juju recognizes the nature of the key and applies it
accordingly. See [Configuring models][models-config] for more information on
how models get configured.

## List of controller keys

This table lists all the controller keys which may be assigned a value.

| Key                        | Type   | Default  | Valid values             | Purpose |
|:---------------------------|--------|----------|--------------------------|:---------|
api-port                     | integer | 17070   |                          | The port to use for connecting to the API
auditing-enabled             | bool   | false    | false/true               | Sets whether the controller will record auditing information
autocert-dns-name            | string |          |                          | Sets the DNS name of the controller. If a client connects to this name, an official certificate will be automatically requested. Connecting to any other host name will use the usual self-generated certificate.
autocert-url                 | string |          |                          | Sets the URL used to obtain official TLS certificates when a client connects to the API. By default, certificates are obtained from LetsEncrypt. A good value for testing is "https://acme-staging.api.letsencrypt.org/directory".
allow-model-access           | bool   |          | false/true               | Sets whether the controller will allow users to connect to models they have been authorized for even when they don't have any access rights to the controller itself.
bootstrap-timeout            | integer | 600     |                          | How long in seconds to wait for a connection to the controller
bootstrap-retry-delay        | integer | 5       |                          | How long in seconds to wait between connection attempts to a controller
bootstrap-address-delay      | integer | 10      |                          | How often in seconds to refresh controller addresses from the API server
ca-cert                      | string |          |                          | The certificate of the CA that signed the controller's CA certificate, in PEM format
controller-uuid              | string |          |                          | The key for the UUID of the controller
identity-public-key          | string |          |                          | Sets the public key of the identity manager
identity-url                 | string |          |                          | Sets the URL of the identity manager
max-logs-age                 | string | 72h      | 72h, etc.                | Sets the maximum age for log entries before they are pruned, in human-readable time format
max-logs-size                | string | 4G       | 400M, 5G, etc.           | Sets the maximum size for the log collection, in human-readable memory format
max-txn-log-size             | string | 10M      | 100M, 1G, etc.           | Sets the maximum size for the capped txn log collection, in human-readable memory format
mongo-memory-profile         | string | low      | low/default              | Sets whether MongoDB uses the least possible memory or the default MongoDB memory profile
set-numa-control-policy      | bool   | false    | false/true               | Sets whether numactl is preferred for running processes with a specific NUMA (Non-Uniform Memory Architecture) scheduling or memory placement policy for multiprocessor systems where memory is divided into multiple memory nodes
state-port                   | integer | 37017   |                          | The port to use for mongo connections


<!-- LINKS -->

[controllers-creating]: ./controllers-creating.html "Creating a controller"
[models-config]: ./models-config.html "Configuring models"
