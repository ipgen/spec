# Internet Protocol Address Generation Specification (IPGen Spec)

Version - 0.0.1

IPGen Spec is a specification describing a method of generating highly unique and reproducible IP addresses.
The core goals of this specification include:

* Generating highly unique IP address (ie. very low probability of collision)
* Ensuring that the generated IP address can be determined upfront (ie. reproducible)
* Using cryptography to achieve the first two goals

The specification consists of two sections:

1. [IPv6 Addresses](#ipv6-addresses) - defines how to generate IPv6 addresses
2. [IPv4 Addresses](#ipv4-addresses) - defines how to generate IPv4 addresses

## Example Use Case

To help clarify what this spec is all about, we will use a very simple example.

A user (let us call him John) wants to deploy their app which needs access to a database management system (let us call it JohnDB) for storing their data. John uses `fd52:f6b0:3162::/48` as his subnet so he wants his IP addresses allocated in that network. JohnDB can be configured to listen for connections on a particular IP address using the environment variable `JOHNDB_ADDRESS`.

In his JohnDB startup script John uses a commandline tool that implements this spec to generate an IP address using his network address `fd52:f6b0:3162::/48` and database name `johndb`. The same startup script then uses the `ip` command to add this address to his network interface so `johndb` can bind to it when it starts up. His stop script deletes the generated IP address off the interface using either the environment variable or regenerating the same IP address using the previous tool.

In his app, John uses a library that implements this specification to generate the same IP address that was generated for `johndb` so he can connect to it. Like the commandline tool, the library will use the same network address (fd52:f6b0:3162::/48) and database name (johndb) to compute the same IP address that was generated for JohnDB. Note that John doesn't have to use a library here if he doesn't want to. He can still use the previous commandline tool to generate the address and expose it as an environment variable as his app starts.

As you can see, John doesn't need to care about the IP address that will be generated. He only needs to decide on a network address to use (something he already does whether he is utilising this spec or not) and a name for the database. If he restarts his database, it will come back up using the same IP address even if he moves it to another host.

## IPv6 Addresses

Here are the steps that tools that implement this spec need to follow to generate IPv6 addresses:

1. Accept at least a network address in CIDR form and the NAME of the thing for which the IP address is being generated. It should be any arbitrary text so users can have the flexibility to use UUIDs or any other identifiers they want.
2. Validate the network address, returning imediately with an error if it isn't.
3. Validate that the network prefix is less that 128, returning imediately with an error if it isn't. In such a case we only have 2 choices, return the same IP address or return an error since this prefix states that this is already a complete IP address. This spec chooses the latter as it helps detect mistakes.
4. Calculate `NETWORK_LENGTH` by dividing the network prefix by 4, discarding any decimals (ie, we are only intrested in the integer). This gives us the total number of characters that we must never touch.
5. Expand the IPv6 address fully and save the first `NETWORK_LENGTH` characters as `NETWORK_HASH`.
6. Calculate `ADDRESS_LENGTH` by subtracting `NETWORK_LENGTH` from 32.
7. Calculate `BLAKE_LEN` by first dividing `ADDRESS_LENGTH` by 2 and then adding the remainder of diving `ADDRESS_LENGTH` by 2.
8. Compute A`DDRESS_HASH` by generating a `blake2b` hash of the `NAME` using `BLAKE_LEN` as output length.
9. Create an `IP_HASH` by first joining `NETWORK_HASH` and `ADDRESS_HASH` in that order and then taking only the first 32 characters of the resulting string.
10. Convert the `IP_HASH` to IPv6 format by placing a colon after every 4 characters but not at the end of the hash.
11. Return the resulting IP address in compressed form. Libraries should return this as a native IPv6 object of their programming language where possible so other tools can work with it easily.

## IPv4 Addresses

IPv4 addresses are calculated using the same method as IPv6 addresses. We simply need to convert the IPv4 address to IPv6 notation, work with it like a normal IPv6 and then convert it back to IPv4 when done.

Here are the steps that tools that implement this spec need to follow to generate IPv4 addresses:

1. Follow steps 1 and 2 in the [IPv6 Addresses](#ipv6-addresses) section.
2. Validate that the network prefix is less that 32, returning imediately with an error if it isn't. In such a case we only have 2 choices, return the same IP address or return an error since this prefix states that this is already a complete IP address. This spec chooses the latter as it helps detect mistakes.
3. Calculate `IP6_PREFIX` by subtracting 32 from 128 and then adding back the IPv4 prefix supplied.
4. Format as IPv6 network address by joining "::" and the supplied IPv4 address and "/" and `IP6_PREFIX` in that order.
5. Follow steps 4 to 10 in the [IPv6 Addresses](#ipv6-addresses) section.
6. Format the resulting IP address in the IP6to4 format (eg ::10.19.28.118/104).
7. Drop the leading "::" and trailing subnet prefix part (ie. "/104" in the example above).
10. Follow step 11 in the [IPv6 Addresses](#ipv6-addresses) section.
