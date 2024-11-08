# How to publish to Maven Central
_as of June 2024; based on material from [Maven Central][central:how-to-publish]_

## Prerequisites
This guide aims at making it as easy as possible for you to publish your first artifact.  
While it's absolutely possible to deviate from this approach, this guide assumes you're in possession of: 
* a GitHub account
* a Linux machine for development (this example has been created with Ubuntu)
* and a Maven-based project waiting to be published
  * basic familiarity with Maven

## Step-by-step guide

### Register at Sonatype central
Hop on over to the [Maven Central repository][central:login], and login via GitHub.
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
1. go to [your account][central:account] on the Central Publisher Portal
1. click the button "Generate User Token"
1. copy the code snippet
1. add it to your existing [Maven settings][maven:settings], or create a new file containing
```xml
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
```
> [!TIP]
> Consider using [Maven password encryption][maven:pw-encryption] for storing these credentials.

### Set up artifact signing
Since you're using a Linux machine, `gpg` should be available.

1. create a key pair:
   ```shell
   gpg --batch --passphrase '' --quick-gen-key "your-username@users.noreply.github.com"
   ```
1. Note down the keyid (the hexadecimal code, usually the second line)
   ```shell
   gpg -k your-username@users.noreply.github.com
   ```
1. Distribute your public key
   ```shell
   gpg --keyserver keys.openpgp.org --send-keys YOUR-KEYID
   ```

> [!NOTE]
> When using GitHub actions to publish, you have to "hand out" both key & passphrase anyway,
> thus we're using an empty passphrase to begin with.  
> If you'd rather want to use a passphrase, use `gpg --gen-key` to create a key.

### Configure your project
Now, your Maven project needs to be configured with a number of plugins.

#### central-publishing-maven-plugin
We'll start with `central-publishing-maven-plugin`, which will upload the artifacts to the Maven Central repository:
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
This will eventually fail because we're still missing a couple of things,
yet your [_Deployments_ section][central:deployments] in the Maven Central repository
should list our attempt to publish something.  
Go ahead and hit the _Drop_ button.

#### Artifact signing, Javadoc & Sources
In order to publish to Maven Central, you need to sign all artifacts, and ship a Javadoc as well as the sources.
Therefore, we need to configure more plugins:
```xml
<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-gpg-plugin</artifactId>
    <version>1.6</version>
    <executions>
      <execution>
        <goals>
          <goal>sign</goal>
        </goals>
      </execution>
    </executions>
  </plugin>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>3.6.3</version>
    <executions>
      <execution>
        <goals>
          <goal>jar</goal>
        </goals>
      </execution>
    </executions>
  </plugin>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>3.3.1</version>
    <executions>
      <execution>
        <goals>
          <goal>jar-no-fork</goal>
        </goals>
      </execution>
    </executions>
  </plugin>
</plugins>
```

Now, let's run the deployment process again:
```shell
mvn -B -ntp deploy
```

> [!NOTE]
> Chances are that generating the Javadoc fails. Fix any errors that occur.

> [!TIP]
> If you happen to have multiple GPG keys, you might have to configure the correct key to use
> (an indicator for that is Sonatype complaining with _"Invalid signature for file"_).  
> There are [several options to do this][gpg:sign];
> using `-Dgpg.keyname=...` when calling Maven is suggested for now:
> when automating the release later on, only one key will be available. 

#### Add missing information
The Maven execution is probably still going to fail, since some information in the POM is missing.
Sonatype requires you to provide
[<sup>_(click here to see what I've added)_</sup>][diff:missing-info]
* a project description
* a project URL
* a license
* developers information
* an SCM URL

### Adapt the version
The only remaining thing to do is adapt the version, since `SNAPSHOT` versions cannot be released.
Go ahead and adapt the version. Now if you run the Maven command again, you should have a successful build.
The [_Deployments_ section][central:deployments] will now show a _VALIDATED_ deployment.

> [!TIP]
> If you think everything's working out, you can go ahead and publish your artifact now! 🎉 

## Automate the release process
For a more automated release process, we want the version numbering handled and have the released code tagged.
We'll use the `maven-release-plugin` for this.

### Maven configuration
We pin the version of the `maven-release-plugin` and overwrite some defaults:
```xml
<pluginManagement>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-release-plugin</artifactId>
      <version>3.1.1</version>
      <configuration>
        <arguments>-ntp</arguments>
        <scmCommentPrefix />
        <tagNameFormat>v@{project.version}</tagNameFormat>
      </configuration>
    </plugin>
  </plugins>
</pluginManagement>
```

And, if not done so already, you need to set the SCM connection in the POM:
```xml
<scm>
  <connection>scm:git:https://github.com/your-username/your-repository.git</connection>
</scm>
```

### Set up the GitHub action
As an execution engine, we'll use GitHub Actions.
So let's create the file `.github/workflows/release.yml`:
```yaml
name: Release
on:
  workflow_dispatch:
jobs:
  release:
    permissions:
      contents: write # to manage the POM's version
    runs-on: ubuntu-latest
    steps:
      - name: Check-out
        uses: actions/checkout@v4
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '21'
          cache: maven
          gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
          server-id: central
          server-username: SONATYPE_USERNAME
          server-password: SONATYPE_TOKEN
      - name: Release with Maven
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          mvn -B -ntp -Dstyle.color=always release:prepare
          VERSION=`cat release.properties | grep scm.tag= | cut -d'=' -f2`
          mvn -B -ntp -Dstyle.color=always release:perform
          echo "Released ${VERSION} 🚀" >> $GITHUB_STEP_SUMMARY
        env:
          GPG_PASSPHRASE: ''
          SONATYPE_USERNAME: ${{ secrets.SONATYPE_USERNAME }}
          SONATYPE_TOKEN: ${{ secrets.SONATYPE_TOKEN }}
```

### Set up secrets
The script refers to 3 secrets that need to be set up before the workflow can successfully run:
the GPG private key for artifact signing, and the Sonatype credentials for publishing.  
To do so,
1. in your GitHub repository, navigate to _Settings/Secrets and variables/Actions_
1. click on _New repository secret"
   1. put `SONATYPE_USERNAME` into the _Name_ field
   1. put your username into the _Value_ field
   1. store the secret by hitting _Add secret_
1. now create a secret for `SONATYPE_TOKEN` following the same steps
1. now create a secret for `GPG_PRIVATE_KEY`
   1. for the _Value_ field, paste the data returned by
      ```shell
      gpg --export-secret-keys --armor YOUR-KEYID
      ```

> [!TIP]
> If your GPG key requires a pass phrase, simply add that as a secret as well, and adapt the `release.yml` accordingly.

### Run the GH action
We're all set to release!  
In your GitHub repository,
go to _Actions_,
click on _Release_ in the left panel,
select _Run workflow_ on the right,
and finally click _Run workflow_ in the pop up.
It'll take a few seconds, then the _workflow run_ will appear.
You may drill into the details, or just wait until the process has finished.

Once again, Maven Central repository's [_Deployments_ section][central:deployments] will show a _VALIDATED_ deployment.

### Final adaptation
For a fully automated release, set the `autoPublish` parameter to `true`. 💪

Now you're all set - go ahead, release your stuff, change the world!

[central:account]: https://central.sonatype.com/account
[central:deployments]: https://central.sonatype.com/publishing/deployments
[central:login]: https://central.sonatype.com/api/auth/login
[central:how-to-publish]: https://central.sonatype.org/register/central-portal/
[diff:missing-info]: https://github.com/sebastiankirsch/how-to-publish-to-central/commit/6df181aa96828122458eaf264a872260d8beb29e#diff-9c5fb3d1b7e3b0f54bc5c4182965c4fe1f9023d449017cece3005d3f90e8e4d8
[gpg:sign]: https://maven.apache.org/plugins/maven-gpg-plugin/sign-mojo.html
[maven:pw-encryption]: https://maven.apache.org/guides/mini/guide-encryption.html
[maven:settings]: https://maven.apache.org/settings.html