# lambda-packaging

[![Maven Central](https://img.shields.io/maven-central/v/io.symphonia/lambda-packaging.svg)](https://search.maven.org/artifact/io.symphonia/lambda-packaging)

A collection of [Maven assembly descriptors](http://maven.apache.org/plugins/maven-assembly-plugin/examples/sharing-descriptors.html) for building [AWS Lambda](https://aws.amazon.com/lambda/) deployment packages.

The `lambda-zip` descriptor supports building `.zip` deployment packages as described in the AWS documentation page, ["Creating a ZIP Deployment Package for a Java Function"](https://docs.aws.amazon.com/lambda/latest/dg/create-deployment-pkg-zip-java.html). The resulting `.zip` file contains compiled class files and resources at the top level, and all of the required dependencies as `.jar` files in the `lib/` directory.

### Advantages of `.zip` over uberjar

This method of packaging Lambda deployment packages has two major advantages over the `uberjar` method:

1. Reduces the amount of time it takes the Lambda runtime to unpack the deployment package (see the AWS Lambda ["Best Practices"](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html) page).
1. Avoids overlaying unpacked dependencies on top of each other, which leads to resource merge conflicts.

### When *not* to use `lambda-packaging`

1. For Lambda functions that have no dependencies other than `aws-lambda-java-core`.
1. In cases where classes need to be [relocated](https://maven.apache.org/plugins/maven-shade-plugin/examples/class-relocation.html) - use the [`maven-shade-plugin`](https://maven.apache.org/plugins/maven-shade-plugin/) instead.

### Usage

Add the following plugin configuration to your `pom.xml` file:

```xml
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.1.0</version>
                <dependencies>
                  <dependency>
                    <groupId>io.symphonia</groupId>
                    <artifactId>lambda-packaging</artifactId>
                    <version>1.0.0</version>
                  </dependency>
                </dependencies>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>lambda-zip</descriptorRef>
                    </descriptorRefs>
                    <appendAssemblyId>false</appendAssemblyId>
                    <finalName>lambda</finalName>
                </configuration>
            </plugin>
            ...
        </plugins>
        ...
    </build>
```

With this configuration in place, running `mvn package` will produce a `lambda.zip` deployment package in the `target/` directory.
