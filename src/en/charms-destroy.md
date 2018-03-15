Title: Removing applications, units, and machines
TODO:  Elaborate on "restore purposes"

# Removing applications, units, and machines

This section looks at how to remove applications, units, and machines. To 
remove a model see the [models documentation][models]. To remove a controller,
see the [controllers documentation][controllers].

## Removing applications

An application can be removed with:

```bash
juju remove-application <application-name>
```

!!! Note: Removing an application which has active relations with another
running application will terminate that relation. Charms are written
to handle this, but be aware that the other application may no 
longer work as expected. To remove relations between deployed applications,
see [Charm relations][charmrelations].

<!-- DUBIOUS CONTENT. IS THERE A REASONABLE BENEFIT TO BE GAINED BY EXPOSING
THESE INTERNAL PROCESSES? MAYBE MOVE IT TO THE TROUBLESHOOTING SECTION?

Internally, this is how Juju processes the removal of an application:

 1. The Juju client tells the controller to destroy all the application's
    units.
 1. The controller signals to the application (charm) that it is going to be
    destroyed.
 1. The charm breaks any relations to its application by calling 
    `relationship-broken` and `relationship-departed`.
 1. The charm calls its 'stop hook' which **should**:
     - Stop the application
     - Remove any files/configuration created during the application lifecycle
     - Prepare any backup(s) of the application that are required for restore 
       purposes.
 1. The application and all its units are then removed.

It can take a while for the application to be completely removed but if
`juju status` reveals that the application is listed as "dying", but also
reports an error state, then the removed application will not go away. See the
'Caveats' section below for how to manage applications stuck in a dying state.

-->


## Removing individual units

It is possible to remove individual units instead of the entire application
(i.e. all the unites):

```bash
juju remove-unit mediawiki/1
```

To remove multiple units:

```bash
juju remove-unit mediawiki/1 mediawiki/2 mediawiki/3 mysql/2
```

In the case that these are the only units running on a machine, unless that 
machine was created manually with `juju add machine`, the machine will also be 
removed.

See section 'Caveats' below for how to manage units in a dying state.

## Removing machines

Juju machines can be removed like this:

```bash
juju remove-machine <number>
```

However, it is not possible to remove a machine which is currently allocated
to a unit. If attempted, this message will be emitted:

```no-highlight
error: no machines were destroyed: machine 3 has unit "mysql/0" assigned
```

By default, when a Juju machine is removed, the backing system, typically a
cloud instance, is also destroyed. The `--keep-instance` option overrides this;
it allows the instance to be left running.

## Caveats

These are caveats which you may encounter while removing items.

### state of *life: dying*

If you have a unit or application that persists in a dying state check to see if
that unit, or any units within the associated application, are in an error 
state. A "removal" is an event within the queue of a unit's lifecycle so when a
unit enters an error state all events within the queue are blocked. To unblock
the unit's error you need to resolve it. Do so like this:

```bash
juju resolved <unit>
```

The above command may need to be repeated to resolve other errors on the unit.

There may be errors on other units caused by the breaking of relations that
occur when removing a unit or application. Therefore also verify that the
associated units are not in an error state and apply the above command to them
if they are.

[charmrelations]: ./charms-relations.html#removing-relations
[controllers]: ./controllers.html
[models]: ./models.html
