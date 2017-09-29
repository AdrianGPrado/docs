Title: Juju users and models
TODO: Stuff on user-added models (ssh key and credentials)
      Check the functionality of admin user access level. This currently
      appears to do nothing (not destroy models, nor backups) 

# Users and models

This section is about understanding models with multiple users.

## Adding models

When an administrator adds a model, by default, the cloud credentials used
throughout the model will be those that the admin used to create the controller
and the SSH keys copied across the model will be those of the controller model.
The administrator can override these defaults with the appropriate command
options.

The model creator becomes, by default, the model owner. However, the creation
process does allow for owner designation.

Examples:

Add model 'mymodel' (in the current controller):

```bash
juju add-model mymodel
```

Add model 'mymodel' and designate user 'tron' as the owner:

```bash
juju add-model --owner=tron mymodel
```

See [Adding a model][addmodel] for details on adding models.


## Models and user access

An administrator can use the `grant` command to grant a user 'read', 'write',
or 'admin' access to any model. 

- `read`: A user can view the state of a model with the `models`,
  `machines` and `status` commands.
- `write`: In addition to 'read' abilities, a user can modify/configure models.
- `admin`: In addition to 'write' abilities, a user can backup/destroy models
  and connect to machines via SSH.

To give 'bob' read-only access to the model 'mymodel', for example, the
administrator would enter the following:

```bash
juju grant bob read mymodel
```

To give 'jim' write access to the same model, the administrator would use the
following:

```bash
juju grant jim write mymodel 
```
See [Users][regularusers] for details on available commands.

!!! Note: Each user has control over naming the models they own. This means
it is possible for two users, `jane` and `claire`, to each have a model with
the same name, `foo`. This could cause difficulty when `claire` needs to access
`jane`'s model. Because of this, it is possible to refer to models
using `<owner>/<model>` in place of just the model name. For example, `claire`
can get the status of the model using `juju status -m jane/foo`.

## Controller access

A controller is a special kind of model that acts as the management node for
each cloud environment. Properly managed access to any controller is critical
to the security and stability of your cloud and all its models. 

In addition to the two levels of user access for models, three further levels
of access are used to manage access to Juju's controllers:

- `login`: the standard access level for any user, enabling them
  to connect to a cloud and access any permitted models.
- `add-model`: in addition to login access, a user is also be permitted
  to add and remove new models.
- `superuser`: grants a user the same permissions as an administrator and complete
  control over the deployed environment. 

The `grant` syntax for controller access is the same as model
access, only without the need to specify a model. With no controller specified,
the current controller will be assumed the target:

```bash
juju grant jim add-model
```

A controller can be specified using the `--controller` argument followed by the
name of the controller:

```bash
juju grant jim add-model --controller admin-lxd
```

The `users` command can be used to list all users registered to a controller, along
with their access levels. The output will look similar to the following:

<!-- JUJUVERSION: 2.0.1-xenial-amd64 -->
<!-- JUJUCOMMAND: juju users -->
```no-highlight
Controller: admin-lxd

Name    Display name  Access     Date created  Last connection
admin*  admin         superuser  2016-11-14    just now
bob                   login      1 hour ago    58 minutes ago
jim                   add-model  2016-11-14    58 minutes ago
```

## Revoke access rights

The 'revoke' command is used to remove a user's access to either a model or a
controller. To revoke 'add-model' controller access for user 'jim', you would
use the following:

```bash
juju revoke jim add-model
```

After a `revoke` command has been issued, a user's access will reduce to the
next lower access level. With the above example, user 'jim' would have have
'add-model' access reduced to 'login' access on the controller. This also means
that if a user has write access to a model, the following command would revoke
both read and write access:

```bash
juju revoke bob read mymodel
```

!!! Note: The admin user has credentials stored in the controller and will
be able to perform functions on any model. However, a regular user who has
been given 'add-model' permissions may need to specify which credential to
use when logging in to a model for the first time. To specify a credential,
run 'juju add-credential'.

[addmodel]: ./models-adding.html
[regularusers]: ./users.html#regular-users
