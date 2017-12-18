Title: Cloud specific Juju troubleshooting


# Cloud specific issues


## Common issues setting up clouds

### juju add-credential

juju add-credential supports adding a credential with a YAML file. The YAML
format is the same that's used to store the credentials on a user's
filesystem. This means that it allows for more than one credential to be
located in the file. However, you can only add one at a time. This often
throws folks that manually create the new credential file. The format must
include a root key of "credentials:".

For example, the following 'azure.yaml' file will fail.

```yaml
azure:
  rickscreds:
      auth-type: service-principal-secret
      application-id: 1234
      application-password: secretpassword
      subscription-id: 12345
      tenant-id: 12345
```

Once the root key is added it will succeed:

```yaml
credentials:
    azure:
      rickstuff:
          auth-type: service-principal-secret
          ...
```

### Upgrading controllers and models

Controllers and their hosted models need to be upgraded whenever Juju is
updated, see [Upgrading Juju software][modelsupgrade] for more details.

But as large model deployments can obviously be complex, you may occasionally
experience problems during an upgrade. These problems can often be mitigated by
restarting the *jujud-*  agents running on the machines within your models.

The following Bash script, for example, connects to each machine in your model
and restarts each agent:

```bash
for i in {0..NUMBER_OF_MACHINES}; do
  juju ssh $i "for app in \$(ls /var/lib/juju/tools); do
    sudo systemctl restart jujud-\$app;
  done";
done
```

For the above example to work, you will need to replace `NUMBER_OF_MACHINES`
with the maximum machine value from `juju status`, and those machines will also
need to be running systemd, rather than running within local LXD containers.

<!--

## MAAS


## OpenStack


## LXD



## Amazon Web Services



## Microsoft Azure



## Google Compute Engine


## Oracle


## Manual

-->

<!-- LINKS -->

[modelsupgrade]: ./models-upgrade.html
