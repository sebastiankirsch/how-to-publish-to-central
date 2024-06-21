# How to publish to Maven Central
_as of June 2024_; based on material found on [maven central](https://central.sonatype.org/register/central-portal/).

## Prerequisites
This guide aims at making it as easy as possible for you to publish your first artifact.  
While it's certainly possible to deviate from this approach, this guide assumes you're in possession of: 
* a GitHub account
* a Linux machine for development
* and a Maven-based project waiting to be published

## Step-by-step guide
Typically, one already has a
### Register at Sonatype central
Hop on over to [maven central repository](https://central.sonatype.com/api/auth/login), and login via GitHub.
This registers the **io.github._yourusername_** namespace for you,
allowing you to publish artifacts matching this namespace.

ℹ If you didn't do so yet, adapt the `<groupId>` of your artifact accordingly.
You might use `io.github.acme` or a subgroups like `io.github.acme.foo`.
Consider renaming the java package names accordingly.

Configure Maven settings
You need to configure the Maven settings such that you can upload artifacts
These settings are usually located at `~/.m2/settings.xml`.

### Set up artifact signing
Since you're using a Linux machine, `gpg` should be available.
ℹ This is a shameless copy & reduction of [central's guide](https://central.sonatype.org/publish/requirements/gpg/#distributing-your-public-key).

Run `gpg` to create a key pair.

Publish the public key.

ℹ Since I'm usually using GitHub actions to publish, and thus I have to "hand out" both key & passphrase anyway,
I'm not setting up a passphrase in the first place. Do what suits your needs best.

### Configure your project
### Releasing
#### Manually
#### via GitHub action