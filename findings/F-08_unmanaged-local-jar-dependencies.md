# F-08: Unmanaged Local JAR Dependencies

## What was observed

Both the CPL and 试玩测试/豆豆赚 codebases include a `lib/` directory containing JAR files referenced via Maven `systemPath` dependencies rather than standard Maven coordinates:

```
lib/fccsplatform-common-crypto-1.0.0.20161108.jar  (11.26 KB)
lib/taobao-sdk-java-auto_1634133349282-20211018.jar (901.32 KB)
```

These JARs are committed directly to the source repository. Maven warns about this during every build:

```
'dependencies.dependency.systemPath' for com.alipay.fc.csplatform:fccsplatform-common-crypto:jar
should not point at files within the project directory
```

The Alipay crypto library dates to November 2016. The Taobao SDK dates to October 2021.

## Why this matters

JARs committed to a repository have no integrity verification mechanism. There is no checksum enforced by a build tool, no signature validation, and no central registry entry to compare against. Anyone with write access to the repository can replace these files with modified versions without any build-time detection.

The Alipay crypto library in particular is responsible for cryptographic operations in payment flows. A tampered version of this library could silently alter payment signature generation or verification behavior.

The 2016 date on the Alipay library means it has received no updates for nearly a decade. Any vulnerabilities discovered in that library since 2016 are present in every build.

## What needs to change

- Replace both JARs with properly managed Maven dependencies using standard coordinates, checksums, and a dependency repository.
- If official Maven coordinates are not available for these libraries, use a private Maven repository with enforced checksum verification.
- Audit the current committed JARs against known-good versions to verify they have not been tampered with.
- Remove the JAR files from git history.
