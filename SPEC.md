# Internet Protocol Address Generation Specification (IPGen Spec)

Version - 0.0.1

IPGen Spec is a specification describing a method of generating highly unique and reproducible IP addresses.
The core goals of this specification include:

* Generating highly unique IP address (i.e. very low probability of collision)
* Ensuring that the generated IP address can be determined upfront (i.e. reproducible)
* Using cryptography to achieve the first two goals

The specification consists of two sections:

1. [IPv6 Addresses](#ipv6-addresses) - defines how to generate IPv6 addresses
2. [IPv4 Addresses](#ipv4-addresses) - defines how to generate IPv4 addresses

## Example Use Case

To help clarify what this spec is all about, we will use a very simple example.

A user (let us call him John) wants to deploy his app which needs access to a database management system (let us call it JohnDB) for storing his data. John uses `fd52:f6b0:3162::/48` as his subnet so he wants his IP addresses allocated in that network. JohnDB can be configured to listen for connections on a particular IP address using the environment variable `JOHNDB_ADDRESS`.

In his JohnDB startup script John uses a command line tool that implements this spec to generate an IP address using his network address `fd52:f6b0:3162::/48` and database name `johndb`. The same startup script then uses the `ip` command to add this address to his network interface so `johndb` can bind to it when it starts up. His stop script deletes the generated IP address off the interface using either the environment variable or regenerating the same IP address using the previous tool.

In his app, John uses a library that implements this specification to generate the same IP address that was generated for `johndb` so he can connect to it. Like the command line tool, the library will use the same network address (fd52:f6b0:3162::/48) and database name (johndb) to compute the same IP address that was generated for JohnDB. Note that John doesn't have to use a library here if he doesn't want to. He can still use the previous command line tool to generate the address and expose it as an environment variable as his app starts.

As you can see, John doesn't need to care about the IP address that will be generated. He only needs to decide on a network address to use (something he already does whether he is utilising this spec or not) and a name for the database. If he restarts his database, it will come back up using the same IP address even if he moves it to another host.

## IPv6 Addresses

Here are the steps that tools that implement this spec need to follow to generate IPv6 addresses:

1. Accept at least a network address in [CIDR notation] and the `NAME` of the thing for which the IP address is being generated. It should be any arbitrary text so users can have the flexibility to use UUIDs or any other identifiers they want.
2. Validate the network address, returning immediately with an error if it isn't valid.
3. Validate that the network prefix is less than or equal to 128, returning immediately with an error if it isn't. If it is equal to 128, return the network address itself.
4. Create `NETWORK_LENGTH` from the network address, representing the length of the network part of the address in bits.
5. Create `NETWORK_HASH` as a bit representation of the network address (`u128`).
6. Compute `ADDRESS_LENGTH` by subtracting `NETWORK_LEN` from 128.
7. Compute `ADDRESS_HASH` by generating a `blake2b` hash of the `NAME` using
   `ADDRESS_LEN`.
   - Since `ADDRESS_LEN` is in bits, you must compute the corresponding amount
   of bytes that need to be generated from a `blake2b` hash by dividing it by 8
   and adding 1 if it is not evenly divisible by 8 (e.g., in Rust: `(ADDRESS_LEN / 8) + (ADDRESS_LEN % 8 != 0)`).
   - Once the correct amount of bytes has been generated, convert the
   collection of bytes to a singular bit representation (`u128`), zeroing out
   any additional bits generated.
       - E.g., If the `ADDRESS_LENGTH` was 7, 1 byte would have to be generated.
       However, this results in an extra bit that would need to be zeroed out.
8. Create `IP_HASH` by calculating the bitwise OR (`|`) of `NETWORK_HASH` and
   `ADDRESS_HASH`.
9. Return the IP address representation of the previous bit representation
   (`IP_HASH`). Libraries should return this as a native IPv6 object of their
   programming language so other tools can work with it easily.

## IPv4 Addresses

IPv4 addresses are calculated using the same method as IPv6 addresses. We simply need to convert the IPv4 address to IPv6 notation, work with it like a normal IPv6 and then convert it back to IPv4 when done.

Here are the steps that tools that implement this spec need to follow to generate IPv4 addresses:

1. Follow steps 1 and 2 in the [IPv6 Addresses](#ipv6-addresses) section.
2. Validate that the network prefix is less than or equal to  32, returning immediately with an error if it isn't. If it is equal to 32, return the network address itself.
3. Calculate `IP6_PREFIX` by subtracting 32 from 128 and then adding back the IPv4 prefix supplied.
4. Convert the IPv4 address to a [Ipv4-mapped IPv6 address](https://en.wikipedia.org/wiki/IPv6#IPv4-mapped_IPv6_addresses)
5. Use the newly created prefix and address to create an IPv6 network address.
6. Follow steps 4 to 10 in the [IPv6 Addresses](#ipv6-addresses) section, using said IPv6 network address.
7. Convert the IPv4-mapped IPv6 address back to an IPv4 address.
8. Return the IP address representation as a native IPv4 object of their
   programming langauge so other tools can work with it easily.

[CIDR notation]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation
