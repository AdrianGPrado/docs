Title: Juju credentials
TODO:  Investigate: shouldn't `model-config` have a default-credential setting?
       Juju 2.4 review: credentials UX rework

# Cloud credentials

In order to access your cloud (with the exception of [LXD][lxd]), Juju will
need to know how to authenticate itself. We use the term “credentials” to 
describe the tokens or keys or secrets used - a set of credentials 
is represented by a _credential name_ that is used to refer to those 
credentials in subsequent commands.


Currently, Juju can use one of three ways to get your credentials for a cloud:

 - Entering credentials interactively on the command line
 - Scanning existing credentials (e.g. environment variables, .novarc files)
 - Importing a user-provided [YAML-formatted][yaml] file.
 

These methods are explained in more detail below.

If you are having difficulty determining the credentials needed for your 
particular cloud, you should find these pages on specific clouds helpful: 

  [Amazon Web Services][aws]
  
  [Microsoft Azure][azure]
  
  [Google Compute Engine][gce]
  
  [Joyent][joyent]
  
  [Rackspace][rackspace]


### Adding credentials via the command line

You can add credentials by running the command:

```bash
juju add-credential <cloud>
```
Juju will then ask for the information it needs. This may vary 
according to the cloud you are using, but will typically look something like
this:

```no-highlight
Enter credential name: carol
Using auth-type "access-key".
Enter access-key: *******
Enter secret-key: *******
Credentials added for cloud aws.
```

Once you have supplied all the information, the credentials will be added.

At present, you will need to manually set one to be the default, if you 
have more than one for a cloud:

```bash
juju set-default-credential <cloud> <credential>
```

Setting a default credential means this will be used by the bootstrap 
command when creating a controller, without having to specify it with
the `--credential` option.


### Scanning existing credentials

Some cloud providers (e.g. AWS, OpenStack) have command line tools which rely on 
environment variables being used to store credentials. If these are in use on 
your system already, or you choose to define them 
([there is extra info here][env]), Juju can import them.

For example, AWS uses the following environment variables (among others):

**AWS_ACCESS_KEY_ID**

**AWS_SECRET_ACCESS_KEY**

If these are already set in your shell (you can echo $AWS_ACCESS_KEY_ID to test)
they can be used by Juju.

To scan your system for credentials Juju can use, run the command:

```bash
juju autoload-credentials
```

This will will ask you whether to store each set of credentials
it finds. Note that this is a 'snapshot' of those stored values - Juju will not 
notice if they change in future.

### Adding credentials from a YAML file.

You can also specify a YAML format file for the credentials. This
file would be similar to, but shorter than this extensive sample, which
we will call mycreds.yaml:

```yaml
credentials:
      aws:
        default-credential: peter
        default-region: us-west-2
        peter:
          auth-type: access-key
          access-key: AKIAIH7SUFMBP455BSQ
          secret-key: HEg5Y1DuGabiLt72LyCLkKnOw+NZkgszh3qIZbWv
        paul:
          auth-type: access-key
          access-key: KAZHUKJHE33P455BSQB
          secret-key: WXg6S5Y1DvwuGt72LwzLKnItt+GRwlkn668sXHqq
      homemaas:
        peter:
          auth-type: oauth1
          maas-oauth: 5weWAsjhe9lnaLKHERNSlke320ah9naldIHnrelks
      homestack:
        default-region: region-a
        peter:
          auth-type: userpass
          password: UberPassK3yz
          tenant-name: appserver
          username: peter
      google:
        peter:
          auth-type: jsonfile
          file: ~/.config/gcloud/application_default_credentials.json
      azure:
        peter:
          auth-type: service-principal-secret
          application-id: niftyapp
          subscription-id: 31fb132e-e774-49dd-adbb-d6a4e966c583
          application-password: UberPassK3yz
      joyent:
        peter:
          auth-type: userpass
          sdc-user: admingal
          sdc-key-id: 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff
          private-key: key (or private-key-path, like `~/.ssh/id_rsa.pub`)
          algorithm: "rsa-sha256"
      vsphere:
        ashley:
          auth-type: userpass
          password: passw0rd
          user: administrator@xyz.com
```

A source file like the above can be added to Juju's list of credentials with 
the command:

```bash
juju add-credential aws -f mycreds.yaml
```

This sample includes all of the default cloud options plus a couple of
special cloud options, MAAS and an OpenStack cloud called `homestack` in
the sample. See [Clouds](./clouds.html).

## Managing credentials

There are several management tasks that can be done related to credentials.

### Listing credentials

You can check what credentials are stored by Juju by running the command:

```bash
juju list-credentials
```

...which will return a list of the known credentials. For example:

<!-- JUJUVERSION: 2.0.0-genericlinux-amd64 -->
<!-- JUJUCOMMAND: juju list-credentials -->
```no-highlight
Cloud      Credentials
aws     bob*, carol
google  wayne
```

The asterisk '*' denotes the default credential, which will be used for the
named cloud unless another is specified.

For YAML output that includes detailed credential information, including
secrets like access keys and passwords:

```bash
juju list-credentials --format yaml --show-secrets
```

The YAML output will be similar to our 'mycreds.yaml' sample above.

### Setting default credentials

You can set the default credential for a cloud:

```bash
juju set-default-credential aws carol
```

If only one credential name exists, it will become the effective default
credential.

### Replacing local credentials

To replace an existing credential locally, edit or create a file, such as
our 'mycreds.yaml' example above, and run:

```bash
juju add-credential aws -f mycreds.yaml --replace
```

This will overwrite existing credential information, so make sure all current
credentials are contained in the file, not just the new or changed one.

Replacing credentials in this way does not update credentials currently in use
(on an existing controller/cloud). See next section for that. The
`add-credential` command is always "pre-bootstrap" in nature.

### Updating credentials

To update credentials currently in use (i.e. cached on the controller) the
`update-credential` command is used. The requirements for using this command,
as compared to the initial `juju bootstrap` (or `juju add-model`) command, are:

 - same cloud name
 - same Juju username (logged in)
 - same credential name

The update is a two-step process. First change the credentials locally with the
`add-credential` command (in conjunction with the `--replace` option) and then
upload those credentials to the controller.

Below, we log in with the correct Juju username (e.g. 'admin'), change the
contents of the credential called 'joe', and then update them on a Google cloud
controller:

```bash
juju add-credential --replace joe
juju update-credential google joe
```

!!! Warning:
    It is not possible to update the credentials if the initial credential name
    is unknown. This restriction will be removed in an upcoming release of
    Juju.

####  Updating credentials with a different Juju user

If you are unable to ascertain the original Juju username then you will need
to use a different one. This implies adding a new credential name, copying over
any authentication material into the old credential name, and finally updating
the credentials. Below we demonstrate this for the Azure cloud:

Add a new temporary credential name (like 'new-credential-name') and gather all
credential sets (new and old):

```bash
juju add-credential azure
juju credentials azure --format yaml --show-secrets > azure-creds.yaml
```

Copy the values of `application-id` and `application-password` from the new set
to the old set.

Then replace the local credentials and upload them to the controller:

```bash
juju add-credential azure -f azure-creds.yaml --replace
juju update-credential azure old-credential-name
```

To be clear, the file `azure-creds.yaml` (used with `add-credential`) should
look similar to:

```no-highlight
Credentials:
  azure:
    new-credential-name:
      auth-type: service-principal-secret
      application-id: foo1
      application-password: foo2
      subscription-id: bar
    old-credential-name:
      auth-type: service-principal-secret
      application-id: foo1
      application-password: foo2
      subscription-id: bar
```

### Removing credentials

If a credential is no longer required, it can be removed:

```bash
juju remove-credential aws bob
```
 






[yaml]: http://www.yaml.org/spec/1.2/spec.html
[lxd]: ./clouds-LXD.html
[aws]: ./help-aws.html
[azure]: ./help-azure.html
[gce]: ./help-google.html
[joyent]: ./help-joyent.html
[rackspace]: ./help-rackspace.html
