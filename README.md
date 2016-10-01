# IPGen Spec

This repository contains a [specification](SPEC.md) for generating unique and reproducible IP addresses. The spec supports both version 4 and version 6 addresses.

## Uniqueness

Any IP address generated for any given thing with a unique name in a particular subnet has a very low probability of colliding with another IP address generated with a different name in the same subnet. This probability will obviously depend on how many IP addresses are available in that particular subnet.

## Reproducibility

Any given thing with a particular name in a particular subnet will always have the same IP address generated for it. Because of this property, any risk of generating a duplicate IP address can be mitigated in advance via simulation.

## Why this spec?

- Manually assigning IP addresses is frustrating and error prone, especially as the network grows bigger
- Randomly generated IP addresses have a higher probability of colliding and no way to determine these collisions upfront
- Centraly managing IP addresses, eg. using service discovery tools or storing in a database, does not only introduce more complexity in deployment, it also introduces overhead (both network and storage) and worse still that central system is a potential point of failure
- This spec introduces a standard way for both assigning IP addresses on your network and utilising them in a reliable manner

## Who controls the spec?

This is an open-source, community-driven project, developed under the [Apache 2.0 license](LICENSE).

## Implementations of the spec

- [Official command line tool](https://github.com/ipgen/cli)
- [Official Rust library](https://github.com/ipgen/rust)
