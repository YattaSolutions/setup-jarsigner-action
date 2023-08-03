# Setup jarsigner Github Action

This action will prepare the workflow runner for signing jars with a code signing certificate using a pkcs11 implementation.

## Inputs

* `auth-certificate` - p12 user authentication certificate (Base64 encoded)
* `auth-password` - password to access the user certificate
* `digicert-apikey` - Key for accessing the cloud service


## Available environment variable
The following environment variable is set for usage in jarsigner build steps
* `PKCS11_CONFIG` - path to pkcs11 config file used in `providerArg`

There are more environment variables set, which are used by the pkcs11 library.

## Example

Include the action in your workflow as follows:
```
name: Build

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-22.04

    steps:
      - uses: actions/checkout@v3

      - name: Setup jar signing
        uses: YattaSolutions/setup-jarsigner-action@v1
        with:
          auth-certificate: ${{ secrets.AUTH_CERTIFICATE }}
          auth-password: ${{ secrets.AUTH_PASSWORD }}
          digicert-apikey: ${{ secrets.DIGICERT_APIKEY }}

      - name: Maven Build
        run: mvn package

```

And adapt the build using jarsigner to use the provided environment variables.
For maven builds the related plugin configuration will be similar to this snippet. Make sure, that the alias is identical to the name given to the certificate (consider using a environment variable here).
```
<pluginManagement>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jarsigner-plugin</artifactId>
      <version>3.0.0</version>
      <executions>
        <execution>
          <id>sign</id>
          <goals>
            <goal>sign</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <keystore>NONE</keystore>
        <alias>code-signing-certificate</alias>
        <storepass>none</storepass>
        <providerClass>sun.security.pkcs11.SunPKCS11</providerClass>
        <storetype>PKCS11</storetype>
        <providerArg>${env.PKCS11_CONFIG}</providerArg>
        <tsa>http://timestamp.digicert.com</tsa>
      </configuration>
    </plugin>
  </plugins>
</pluginManagement>
<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jarsigner-plugin</artifactId>
  </plugin>
</plugins>
```
Now the build will use jarsigner to sign all jars in the sign goal using the provided certificate.

# License

The scripts and documentation in this repository are released under the [EPL-2.0](LICENSE)
