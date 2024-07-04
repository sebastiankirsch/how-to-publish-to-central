# How to publish to Maven Central
_as of June 2024_; based on material from [maven central](https://central.sonatype.org/register/central-portal/)

## Prerequisites
This guide aims at making it as easy as possible for you to publish your first artifact.  
While it's absolutely possible to deviate from this approach, this guide assumes you're in possession of: 
* a GitHub account
* a Linux machine for development (this example has been created with Ubuntu)
* and a Maven-based project waiting to be published
  * basic familiarity with Maven

## Step-by-step guide

### Register at Sonatype central
Hop on over to [maven central repository](https://central.sonatype.com/api/auth/login), and login via GitHub.
This registers the **io.github._your-username_** namespace for you,
allowing you to publish artifacts matching this namespace.

> [!IMPORTANT]  
> If you didn't do so yet, adapt the `<groupId>` of your artifact accordingly.
> You might use `io.github.your-username` or subgroups like `io.github.your-username.foo`.  
> Consider renaming the package names accordingly.

#### Configure Maven settings
For the artifact publishing, Maven needs access to your credentials.
Since you don't want to share your credentials by listing them in the project's POM,
these are put into the Maven settings, which are usually located at `~/.m2/settings.xml`.

As credentials, you'll be using a user token:
1. go to [your account](https://central.sonatype.com/account) on the Central Publisher Portal
1. click the button "Generate User Token"
1. copy the code snippet
1. add it to your existing Maven settings, or create a new file containing
    
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

1. create a key pair:

       gpg --batch --passphrase '' --quick-gen-key "your-username@users.noreply.github.com"
1. Note down the keyid (the hexadecimal code, usually the second line)

       gpg -k your-username@users.noreply.github.com
1. Distribute your public key

       gpg --keyserver keys.openpgp.org --send-keys YOUR-KEYID

> [!NOTE]
> When using GitHub actions to publish, you have to "hand out" both key & passphrase anyway,
> thus we're using an empty passphrase to begin with.  
> If you'd rather want to use a passphrase, use `gpg --gen-key` to create a key.

### Configure your project
Now, your Maven project needs to be configured with a number of plugins.
Personally, I like to configure plugins only 

#### central-publishing-maven-plugin
We'll start with `central-publishing-maven-plugin`, which will upload the artifacts to the maven central repository: 

```xml
<plugin>
  <groupId>org.sonatype.central</groupId>
  <artifactId>central-publishing-maven-plugin</artifactId>
  <version>0.5.0</version>
  <extensions>true</extensions>
  <configuration>
    <autoPublish>false</autoPublish>
    <deploymentName>${project.artifactId}:${project.version}</deploymentName>
    <waitUntil>validated</waitUntil>
  </configuration>
</plugin>
```

With this in place, we can try out the publish process!

```shell
mvn -B -ntp deploy
```

#### maven-release-plugin
You can safely 

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-release-plugin</artifactId>
  <version>3.0.1</version>
  <configuration>
    <releaseProfiles>release</releaseProfiles>
    <tagNameFormat>v@{project.version}</tagNameFormat>
  </configuration>
</plugin>
```

### Releasing
#### Manually
#### via GitHub action
