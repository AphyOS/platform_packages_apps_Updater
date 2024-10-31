> [!NOTE]
> This repository is a fork, a public mirror and as such will be force-pushed to.

# AphyOS System Updater Application (OTA Client)

This seamless Updater application is intended for use in Apostrophy's infrastructure for devices
running a version of AphyOS.

It is a fork of
[Graphene's Updater application](https://github.com/GrapheneOS/platform_packages_apps_Updater/).

It differs from upstream in the following aspects;

*   Devices create a (pseudo-)random ID for communication with the OTA server.

*   When devices poll for the availability of an update, they include their current build version
    (string), as well as their readiness (boolean).

*   Depending on a roll-out policy, the OTA server can now decide the pace and conditions under
    which to make updates available to devices.

*   To enable an OTA server-side decision to pause or stop a roll-out plan, devices report their
    update progress along the way.

## (Pseudo-)random ID?

The ID is pseudo-random from basically the first 15 characters of a locally generated UUIDv4
string -- which isn't truly random, but random enough for its purpose.

This ID can be refreshed at the user's discretion, by cleaning the storage of the "System Updater"
application. The only relation to a user and an ID is available on the device itself, and the OTA
server only knows about the ID.

## Readiness?

Provided conditions of the device and settings the user controls, the device may deem itself ready,
or not, based on;

*   minimum battery charge level,
*   a requirement to be on a charger (and charging),
*   storage capacity remaining, and/or
*   network connectivity restrictions.

It is expected that in larger fleets of devices, a significant proportion of devices would report
to be "not ready". Enabling the device to indicate its readiness to the OTA server allows the OTA
server to re-assign membership to roll-out pools.

## But Why Tho?

The core functionality pursued here is roll-out plans, controlled on the OTA server side, with some
of the following features;

*   Applying OTA updates to select devices (ex. developer's devices) in a true representation of
    end-to-end real-world device and OTA server environment, including critical path validation.

*   Certain users (read: their devices) may be associated with an "early adopters" group [^1].

*   It is useful to be able to pause or stop a roll-out based on the rate of devices successfully
    upgrading.

*   Roll-out plans prevent bricking devices (July 19th, 2024).

*   Roll-out plans prevent congestion.

While such functionality would typically be associated with "managed devices", roll-out plans are
generally considered good practice, and the use of an application-specific ID used purely for the
reporting of progress in the OTA transaction (or lack thereof) makes this possible.

## OTA Server Workflow

On the OTA server side, we create builds (versions of the operating system). Devices running those
builds are record (ID, build ID) and thus counted.

An upgrade is created for the *source build* being upgraded to the *target build* by providing the
incremental OTA package, which will initially be inactive (thus not commonly available). A regular
alpha/beta/stable channel can be assigned at this point.

A few developers or quality assurance engineers open the updater on their device and record the
device's ID. The OTA server can now associate the inactive update with the specific devices.

This association of an inactive update with a specific device makes the update available to that
device, especially useful in end-to-end real-world critical path testing.

When an update is released or made generally available, the OTA server (intends to [^2]) first
select a small, finite subset of the eligible devices, and issues the update to that subset. The
expectation is then, that these devices return with a positive report, affirming successful
installation (awaiting reboot). It then expects that reboot to be reported as successful, running
with the *target build*, within a finite and measurable period of time, meanwhile halting the
roll-out (to further devices). Should 100% of devices return OK (say, 10/10), then the size of the
roll-out pool can be increased to 500, then 10000, etc.

[^1]: The channels **alpha** and **beta** (vs. **stable**) help, and are at the user's discretion
    (and thus already available). A further association of users (read: employees' devices) and
    groups (such as "dev", "support", "power users", "everyone", "management", etc.) benefits
    corporate deployments, but without serial numbers and IMEIs.

[^2]: This is not currently implemented, and may be based on the first few devices that report
    themselves "ready". Such roll-out approach could also be made optional in its entirety, and
    maybe the sizes of roll-out pools (or "the schedule") become configurable.
