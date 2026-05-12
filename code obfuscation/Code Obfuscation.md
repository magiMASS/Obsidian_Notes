### Maven setup


To setup the maven repo for both clients and devs Refer to this document ->[[Maven_Setup_Guide_Minimal_Obsidian (1)]]

### Artifactory deployment (NEXUS)


For deployment of nexus artifactory service  refer here -> [[Deployment for Artifactory]]

# core project build pom

```xml
	<build>
		<plugins>
			<plugin>
				<groupId>com.github.wvengen</groupId>
				<artifactId>proguard-maven-plugin</artifactId>
				<version>2.7.0</version>

				<executions>
					<execution>
						<id>obfuscate</id>
						<phase>process-classes</phase>
						<goals>
							<goal>proguard</goal>
						</goals>
					</execution>
				</executions>

				<configuration>

					<!-- IMPORTANT: use compiled classes directory -->
					<injar>classes</injar>
					<outjar>classes</outjar>

					<proguardInclude>
						${project.basedir}/proguard.pro
					</proguardInclude>

				</configuration>
			</plugin>


			<!-- 3️⃣ Build normal jar from obfuscated classes -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
```




using proguard pro plugins and proguard rules we can obfuscate 