# Changing Maven to Use Current DateTime as Version and Include in Artifact

To configure your Maven build to use the current date/time as the version and include this in the build artifact, you'll need to:

1. Dynamically set the version during the build
2. Include the timestamp in the final artifact name

Here's how to implement this:

## Option 1: Using Maven Properties and Build Number Plugin

Add this to your `pom.xml`:

```xml
<project>
    <!-- ... other project configuration ... -->
    
    <properties>
        <timestamp>${maven.build.timestamp}</timestamp>
        <maven.build.timestamp.format>yyyyMMdd-HHmmss</maven.build.timestamp.format>
    </properties>
    
    <version>${timestamp}</version>
    
    <build>
        <finalName>${project.artifactId}-${timestamp}</finalName>
        
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>buildnumber-maven-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <phase>validate</phase>
                        <goals>
                            <goal>create</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## Option 2: Using Maven Profiles (More Flexible)

```xml
<project>
    <!-- ... other project configuration ... -->
    
    <profiles>
        <profile>
            <id>timestamp-version</id>
            <activation>
                <property>
                    <name>!version</name>
                </property>
            </activation>
            <properties>
                <version.prefix>1.0.0</version.prefix>
                <timestamp>${maven.build.timestamp}</timestamp>
                <maven.build.timestamp.format>yyyyMMdd-HHmmss</maven.build.timestamp.format>
                <project.version>${version.prefix}-${timestamp}</project.version>
            </properties>
            <build>
                <finalName>${project.artifactId}-${project.version}</finalName>
            </build>
        </profile>
    </profiles>
    
    <!-- Default version if not using timestamp profile -->
    <version>${project.version}</version>
</project>
```

## Option 3: Using Maven Filtering and Resource Files

If you want the timestamp to appear inside your application (like in a properties file):

1. Create a `version.properties` file in `src/main/resources`:
```
build.version=${project.version}
build.timestamp=${timestamp}
```

2. Configure filtering in your `pom.xml`:
```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>3.3.2</version>
            <configuration>
                <warName>${project.artifactId}-${timestamp}</warName>
            </configuration>
        </plugin>
    </plugins>
</build>
```

## Accessing the Version in Java Code

You can then access the version in your Java code by:

```java
public class VersionInfo {
    public static String getVersion() {
        try (InputStream is = VersionInfo.class.getResourceAsStream("/version.properties")) {
            Properties props = new Properties();
            props.load(is);
            return props.getProperty("build.version") + " (" + 
                   props.getProperty("build.timestamp") + ")";
        } catch (IOException e) {
            return "UNKNOWN";
        }
    }
}
```

## Building with the Timestamp Version

When you build with:
```
mvn clean package
```

Your artifact will be named something like:
```
myapp-20240609-142305.jar
```

Note that using dynamic versions can cause issues with Maven's dependency resolution if this artifact is used as a dependency in other projects. This approach is best suited for final deployment artifacts rather than library projects.
