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
This registers the **io.github._your-username_** namespace for you,
allowing you to publish artifacts matching this namespace.

> [!IMPORTANT]  
> If you didn't do so yet, adapt the `<groupId>` of your artifact accordingly.
> You might use `io.github.your-username` or subgroups like `io.github.your-username.foo`.  
> Consider renaming the java package names accordingly.

#### Configure Maven settings
For the artifact publishing, Maven needs access to your credentials.
Since you don't want to share your credentials by listing them in the project's POM,
these are put into the Maven settings, which are usually located at `~/.m2/settings.xml`.

As credentials, you'll be using a user token:
1. go to [your account](https://central.sonatype.com/account) on the Central Publisher Portal
2. click the button "Generate User Token"
3. copy the code snippet
4. add it to your existing Maven settings, or create a new file with
    <?xml version="1.0" encoding="UTF-8"?>
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <servers>
        <server>
          <id>central</id>
          <username>...</username>
          <password>...</password>
        </server>
      </servers>
   </settings>

### Set up artifact signing
Since you're using a Linux machine, `gpg` should be available.

Run `gpg` to create a key pair.

Publish the public key.

> [!TIP]
> Since I'm usually using GitHub actions to publish, and thus I have to "hand out" both key & passphrase anyway,
> I'm not setting up a passphrase in the first place. Do what suits your needs best.

### Configure your project
### Releasing
#### Manually
#### via GitHub action
