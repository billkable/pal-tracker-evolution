# PAL Tracker - Cloud Native application evolution

This project demonstrates the evolution of the PAL Tracker codebase in the
*Master Class for Java Developers* course to the *PAL Tracker Distributed*
codebase and beyond.

The objective is to show various steps in design and evolution of a
Cloud Native codebase,
as well as the associated challenges.

## Release 0.0.1

This release is a simple demo of PAL Tracker application to show basic
backlog tracking functionality to declared projects.

It is not intended for production use.

## Release 1.0.0

This release makes the backlog functionality "production-ready" with
user authentication and update of a story.

## Release 1.1.0

This releae adds allocation tracking so managers can allocation their
employees (workers) to projects.

## Release 1.2.0

This release adds time tracking so managers can tracker their
employee time against a project

## Relase 1.2.6

Application operators determine they need to tune the app to better
scale it out.

## Release 1.2.10

Application operators notice the resource allocation is not efficient
for the monolithic tracker application:

1.  Registration server is heavy on memory usage,
    but not heavily used.

1.  Backlog server is very lightweight per unit work,
    but verify heavily used.
    It makes sense to split out to a separate service.

## Interlude

PAL Tracker application runs for awhile without much changes.
Minor updates and bugfixes

## Release 2.0.0

Product managers of PAL Tracker have potential external
customers that are interested in paying for the Backlog
service as a SaaS.

This means massive scaling the Backlog and Registration
services.

The customers are not interested in the Allocations and
Timesheets services.
Internally these services not frequently used.

Scale up the Backlog and Registration servers.

## Release 2.0.8

PAL Tracker is successfully servicing thousands of
users.

The platform operators notice a lot of allocated
memory by the Registration service,
although the load is low.

How can you reduce the footprint?

1.  Scale down instances.
1.  Scale down memory on a per-instance allocation.
1.  Split out the Timesheets and Allocation functionality
    into separate services.

## Release 2.1.1

During release 2.1.0,
external customers want capability to integrate to 3rd
party SSO services,
rather than rely on the Registration service auth server.

The app team integrates support for Auth0, Google and
Facebook.

They do (what they believe to be) sufficient testing,
but during the first rollout notice the SLO's for
login latency cannot be met.
The side effect is increased thread usage in the
Registration server that at peak times hang the
Registration servers.
There is not sufficient capacity left on the platform
to add registration servers.

What can we do to help mitigate the situation?

1.  Evaluate timeouts on calls of the backlog server
    to the registration server.
1.  Protect the backlog server on calls to the
    registration server with bulkheads and circuit
    breakers.

## Release 2.1.2

The circuit breaker fix in R2.1.1 is clearly a stop-gap
solution to stabilize the system.
But not a long term solution.

There are options to consider:

### Increase available registration app capacity

This requires adding instances of the registration
server,
but that requires more capacity,
which costs more hardware resources available on 
the platform,
which means higher cost for same number of users.

### Reduce amount of work on the registration app

Are all of the calls between the Backlog server to the
Registration server necessary?

Active state of projects are generally set during
creation of the project,
and are only deactivated after a project is closed.

Realtime access to the project state is likely not
necessary,
and using alternate architecture that accomodate
*eventual consistency* may be acceptable.

Potential solutions to consider:

1.  Provide a "project status" services whose sole
    purpose is to provide the project state as a
    backing service (perhaps a cache).
    The cache state must still be populated from
    the registration app.

1.  Add project status state persistence in the
    Backlog server (an aggregate root),
    and update it via an event published by the
    Registration server.

The tradeoffs include some complication in architecture:

1.  A cache or another backing service.

1.  A message broker,
    publisher and listener processes to propagate
    events.

1.  Eventual consistency - either option requires
    some delay between the actual project activation
    versus the backlog server having access to the
    active project.

Pick either option depending on what is better
supported by your platform and organization at
a more attraction resource and support cost.
