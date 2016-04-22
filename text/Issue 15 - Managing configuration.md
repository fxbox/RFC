# What is the problem we're trying to solve ?

Currently we manage configuration using a ConfigService that takes its values either from the command line
and/or a configuration file.

But we'll want to expose configurations to the user through a web interface. Moreover some configuration
might be dynamic (eg: door lock's codes, which luminosities mean day and night).

# Solutions

## Expose the configuration through the Taxonomy

Each configuration is likely a Read/Write value (we could imagine a read-only value for some though, but likely
never write-only values).

So it makes sense to expose these configurations through a Getter/Setter (or whatever we come up with the
decentralised taxonomy) that would get a new property (eg: `purpose`) with value `Config` (a normal Getter/Setter could have
a type `User`).

Configuration values could be:
* coming from the adapters, either dynamically or statically
* coming from foxbox itself
