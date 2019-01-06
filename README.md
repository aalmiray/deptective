# Deptective

🕵️ Deptective is a plug-in for the Java compiler (_javac_) that validates the dependencies
amongst a project's packages against a description of allowed dependences
and fails the compilation when detecting any unintentional dependencies.

* [Requirements](#requirements)
* [Usage](#usage)
   * [Structure of <em>deptective.json</em>](#structure-of-deptectivejson)
   * [Configuring the Java compiler](#configuring-the-java-compiler)
   * [Configuration Options](#configuration-options)
   * [Obtaining Deptective via Jitpack](#obtaining-deptective-via-jitpack)
* [Contributing and Development](#contributing-and-development)
   * [IDE Set-Up](#ide-set-up)
* [Related Work](#related-work)
* [License](#license)

## Requirements

🕵 JDK 8 or later is needed to run Deptective.

The plug-in is specific to _javac_, i.e. the compiler coming with the JDK, it does not work with other compilers such as the _Eclipse Batch Compiler_ (_ecj_).
Support for _ecj_ may be added [later on](https://github.com/moditect/deptective/issues/2).

## Usage

🕵 Deptective is configured through a file _deptective.json_ which describes the allowed dependencies amongst the project's packages.

### Structure of _deptective.json_

🕵 The _deptective.json_ file is structured like this:

```
{
    "packages" : [
        {
            "name" : "com.example.foo",
            "reads" : [
                "com.example.bar",
                "com.example.baz"
            ]
        },
        {
            "name" : "com.example.bar",
            "reads" : [
                "com.example.baz"
            ]
        }
    ],
    "whitelisted" : [
        "java.util*",
        "java.swing*"
    ]
}
```

`packages` is a list of `Package` objects.
The `Package` object has a `name` property (fully-qualified name of the described package)
and a `reads` property (list of strings representing the fully-qualified names of other packages read by the given package).

`whitelisted` is a list of strings representing whitelisted packages,
i.e. packages that always can be read by any other package.
The `*` character can be used as a wildcard, so e.g. `java.util*` will whitelist the packages `java.util`, `java.util.concurrent` etc.

_Note:_ access to the package `java.lang` is always allowed.

Place the configuration file in the root of your source directory (e.g. _src/main/java_ for Maven projects)
or on the classpath at _META-INF/deptective.json_ (e.g. _META-INF/src/main/resources/deptective.json_ for Maven projects).
Alternatively you can specify the location of the config file using the `-Adeptective.configfile` option (see below).

### Configuring the Java compiler

🕵 Add _deptective-javac-plugin-1.0-SNAPSHOT.jar_ to your project's annotation processor path
and specify the option `-Xplugin:Deptective` when invoking _javac_.
When using Maven, the following configuration can be used:

```
...
<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <showWarnings>true</showWarnings>
                <compilerArgs>
                    <arg>-Xplugin:Deptective</arg>
                    <!-- specify options like so -->
                    <!-- <arg>-Adeptective.reportingpolicy=WARN</arg> -->
                </compilerArgs>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.moditect.deptective</groupId>
                        <artifactId>deptective-javac-plugin</artifactId>
                        <version>1.0-SNAPSHOT</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
...
```

See _integration-test/pom.xml_ for a complete example.

### Configuration Options

🕵 The following options can be provided when running the plug-in:

* `-Adeptective.config_file=path/to/deptective.json`: Path of the configuration file in the file system
* `-Adeptective.reporting_policy=(ERROR|WARN)`: Whether to fail the build or just raise a warning when spotting any illegal package dependencies (defaults to `ERROR`; make sure to set `<showWarnings>` to `true` when using the plug-in via Maven)
* `-Adeptective.unconfigured_package_reporting_policy=(ERROR|WARN)`: Whether to fail the build or just raise a warning when detecting a package that's not configured in the config file (defaults to `WARN`)
* `-Adeptective.mode=(ANALYZE|VALIDATE|VISUALIZE)`: Whether the plug-in should validate the packages of the compiled package against the _deptective.json_ file (`VALIDATE`), whether the plug-in should visualize the configured _deptective.json_ file in DOT/GraphViz format (`VISUALIZE`) or whether it should generate a template for that file based on the current actual package relationships (`ANALYZE`). The latter can be useful when introducing Deptective into an existing code base where writing the configuration from scratch might be too tedious. Generating the configuration from the current "is" state and iteratively refining it into an intended target state can be a useful approach in that case. The JSON/DOT files created by `ANALYZE` and `VISUALIZE` are created in the compiler's class output path, e.g. _target/classes_ in case of Maven. Defaults to `VALIDATE`
* `Adeptective.whitelisted=...`: A comma-separated list of whitelist package patterns which will be applied in `ANALYZE` mode. Any reference to a whitelisted package will then not be added to the `reads` section of the referencing package in the generated descriptor template.
The special value `*ALL_EXTERNAL*` can be used to automatically whitelist all packages which are not part of the current compilation (i.e. packages from dependencies). This can be useful if you're only interested in managing the relationships amongst the current project's packages themselves but not the relationships to external packages.

### Obtaining Deptective via Jitpack

🕵 Deptective is not yet available in Maven Central.
For the time being, you can obtain the latest snapshot JARs via [Jitpack](https://jitpack.io/).
Add the following repository to your project's _pom.xml_ or your Maven _settings.xml_ file:

```
<repositories>
    <repository>
        <id>jitpack</id>
        <name>Jitpack</name>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
```

Then reference the Deptective JAR using the GAV coordinates `com.github.moditect.deptective:deptective-javac-plugin:master-SNAPSHOT`.

See _jitpack-example/pom.xml_ for a complete example.

## Contributing and Development

🕵 In order to build Deptective, [OpenJDK 11](https://openjdk.java.net/projects/jdk/11/) or later and Apache Maven 3.x must be installed.
Then obtain the source code from GitHub and build it like so:

```
git clone https://github.com/moditect/deptective.git
cd deptective
mvn clean install
```

Your contributions to Deptective in form of [pull requests](https://help.github.com/articles/about-pull-requests/) are very welcomed.
Before working on larger changes, it's recommended to get in touch first to make sure there's agreement on the feature and design.

### IDE Set-Up

🕵 To work on the code base in Eclipse, please follow this steps:

1. Run Eclipse with (at least) Version 2018-12/4.10.0 and make sure it runs with OpenJDK 11
2. In Eclipse, register an OpenJDK 11 instance ("Preferences" -> "Java" -> "Installed JREs") if not already there
3. Then run "File" -> "Import..." -> "Maven" -> "Existing Maven Projects" and select the root folder of this repository.
4. After importing the project, make sure that Java 11 is on the build path of the _javac-plugin_ module
(right-click on that module, then "Properties" -> "Java Build Path" -> "Libraries").

## Related Work

🕵 Different projects exist that analyze Java package dependencies, validate and/or produce metrics on them.
I'm not aware of any tool though that provides instantaneous feedback about any unwanted dependencies right during compilation.
Some related tools are:

* [ArchUnit](https://www.archunit.org/) aims at enforcing architectures described in a Java DSL.
In contrast to Deptective it is not executed during compilation but via (JUnit) tests.
* [JDepend](https://github.com/clarkware/jdepend) analyzes Java packages and produces metrics on them.
* The Eclipse Java compiler allows to put access restrictions in place but they can only be used to limit access to types/packages in other JARs on the classpath, not to packages of the current compilation unit itself
* [code-assert](https://github.com/nidi3/code-assert) allows to enforce dependency rules described in a Java DSL.
Similarly to ArchUnit, these rules are executed as unit tests.
* [Macker](https://innig.net/macker/) enforces architectural rules at build time via dedicated build tool plug-ins. (not actively maintained)

## License

🕵 Deptective is licensed under the Apache License version 2.0.
