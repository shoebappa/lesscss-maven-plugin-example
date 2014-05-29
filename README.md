# lesscss-maven-plugin AEM / Bootstrap Example

This is a Maven example of compiling Less CSS Resources as part of a build.  This example will build [Bootstrap](http://getbootstrap.com/).  This has some Adobe Experience Manager (AEM) specific plugins because that was the particular use case, but the general plugins should work for any Maven-centric build.

The main benefit of using this plugin versus using the ootb AEM Less Clientlib support is that the Less files are compiled to CSS files at build time giving feedback of syntax and other compile errors earlier on in the development process.  You include the compile CSS files your Client Libraries.  The Maven Less solution will let you use newer versions of Less than are included by AEM.  It should also avoid some of the problems that have been run into with includes in the ootb support through clientlibs.

The first step defines the `generated-resources` directory of the build `target` directory as a source of project resources.  This also defines the `jcr_root` as another sources of resources.  The `generated-resources` directory will be used as a staging area, where the less files are copied, and then compiled.  Maven combines the resources back later.

```
        <resources>
            <resource>
                <directory>src/main/content/jcr_root</directory>
                <filtering>false</filtering>
                <excludes>
                    <exclude>**/.vlt</exclude>
                    <exclude>**/.vltignore</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>${project.build.directory}/generated-resources</directory>
            </resource>
        </resources>
```

The second step uses `the maven-resources-plugin` copies any files from `${basedir}/src/main/content/jcr_root/etc` into the `generated-resources` target.

```
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <includeEmptyDirs>true</includeEmptyDirs>
                </configuration>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>initialize</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/generated-resources/etc</outputDirectory>
                            <resources>          
                                <resource>
                                    <directory>${basedir}/src/main/content/jcr_root/etc</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

The third step uses the `lesscss-maven-plugin` to compile any less files in the `generated-resources` directory.  Because not all of the less files in bootstrap are individually compilable (they are each included in a specific order), I couldn't use a wild-card.  But you can have multiple `<include>` definitions, including wildcards such as `<include>**/*.less</include>` which would try to compile any 

```
            <plugin>
                <groupId>org.lesscss</groupId>
                <artifactId>lesscss-maven-plugin</artifactId>
                <version>1.7.0.1.0</version>
                <executions>
                    <execution>
                        <id>generate-resources</id>
                        <phase>generate-resources</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <sourceDirectory>${project.build.directory}/generated-resources</sourceDirectory>
                            <outputDirectory>${project.build.directory}/generated-resources</outputDirectory>
                            <compress>false</compress>
                            <includes>
                                <include>etc/designs/r2i/libs/bootstrap/bootstrap.less</include>
                            </includes>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

If using the `content-package-maven-plugin` to build a package for deployment the default will just pull the resources from the original files, ignoring the `generated-resources` and thus the compiled less files.  To get the plugin to look at the Maven merged resources, this line was added to the configuration: `<builtContentDirectory>${project.build.directory}/classes</builtContentDirectory>`