# WKND Tutorial Site Notes

## Part 1: Project Setup
* https://helpx.adobe.com/experience-manager/kt/sites/using/getting-started-wknd-tutorial-develop/part1.html
* Used Lazybones to initialize the sample project
* Maven is used to install the dependencies for the project

### Maven
* Maven is a tool for building and managing Java-based project
* Maven allows a project to build using its project object model (POM) and a set of plugins that are shared by all projects using Maven, providing a uniform build system.
* https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html
* https://maven.apache.org/what-is-maven.html

### OSGi
* Open Services Gateway Initiative
* Defines an architecture for modular application development.
* OSGi container implementations such as Knopflerfish, Equinox, and Apache Felix allow you to break your application into multiple modules and thus more easily manage cross-dependencies between them.
* https://www.javaworld.com/article/2077837/application-development/java-se-hello-osgi-part-1-bundles-for-beginners.html


### AEM Project Structure
* Three Main Components
  1. Parent POM
    * Deploys Maven Modules and manages dependency versions
  2. core
    * OSGi Bundle
  3. ui.apps
    * AEM Content Package

#### Parent POM
* POM = Project Object Model, written in XML
* AEM parent POM defines several global properties
  * CRX host, port, username password etc.
  * Typically default to developer standards
    * localhost, admin/admin creds etc.
  * Can overwrite these at the command line when you invoke maven
    * `mvn -PautoInstallPackage clean install -Dcrx.host=production.hostname`
* POM also includes a `<dependencyManagement>` section
  * Section lists all dependencies and versions of APIs that will be used in the project
  * Parent POM should be the only one to include versions. Submodules should not include version info.
  * Will typically include the AEM Uber Jar as a dependency
    * Bundles up all of the AEM APIs
* POM will build two submodules: core and ui.apps. More submodules can be added later.

* Partial Example POM
```xml
<properties>
        <crx.host>localhost</crx.host>
        <crx.port>4502</crx.port>
        <crx.username>admin</crx.username>
        <crx.password>admin</crx.password>
        <publish.crx.host>localhost</publish.crx.host>
        <publish.crx.port>4503</publish.crx.port>
        <publish.crx.username>admin</publish.crx.username>
        <publish.crx.password>admin</publish.crx.password>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
...
<dependencyManagement>
        ...
         <dependency>
                <groupId>com.adobe.aem</groupId>
                <artifactId>uber-jar</artifactId>
                <version>6.3.0</version>
                <scope>provided</scope>
                <classifier>apis</classifier>
            </dependency>
            ...
</dependencyManagement>
...
<modules>
    <module>core</module>
    <module>ui.apps</module>
</modules>
```

#### Core Module
* Includes the Java code for the implementation
* Module packages up the Java code and deploys it to AEM as a OSGi bundle
* Core modules has it's own POM file
* POM file is responsible for compiling Java code into a OSGi bundle that can be recognized by AEM's OSGi container.
* Sling Models are defined in the core module POM file
* Maven Sling plugin allows the core bundle to be deployed to AEM directly leveraging the autoInstallBundle in the parent POM
* `slingURL` defines the location to deploy the bundle
* The OSGi bundle that gets created by Maven when using the core POM is a JAR that gets deployed to an AEM repository as an embedded part of the ui.apps module

* Example core POM

```xml
<plugin>
   <groupId>org.apache.felix</groupId>
   <artifactId>maven-bundle-plugin</artifactId>
   <extensions>true</extensions>
   <configuration>
       <instructions>
           <Bundle-SymbolicName>com.adobe.aem.guides.wknd-sites-guide</Bundle-SymbolicName>
           <Sling-Model-Packages>
             com.adobe.aem.guides.wknd.models
           </Sling-Model-Packages>
       </instructions>
   </configuration>
</plugin>

<plugin>
    <groupId>org.apache.sling</groupId>
    <artifactId>maven-sling-plugin</artifactId>
    <configuration
        <slingUrl>http://${crx.host}:${crx.port}/apps/wknd/install</slingUrl>
        <usePut>true</usePut>
    </configuration>
</plugin>
```

#### ui.apps Module
* Maven module that includes all of the front-end code for the site.
* Includes CSS/JS stored in an AEM format called clientlibs
* Also includes HTL (HTML Template Language) scripts for rendering dynamic HTML
* Think of ui.apps as a map to the structure in the JCR that is in a format that can be stored on a filesystem and source controlled
* Content Package Maven plugin used to compile the contents of ui.apps module into an AEM module that can be deployed to AEM
  * In the ui.apps POM file
* POM File configuration items
  * Package Manager Group, `<group>`: Defines the package manager group
  * `<filterSource>`: Points to the location of filter.xml, which defines JCR paths to include in the package
  * `<embedded>`: Includes the core bundle as part of ui.apps and where it should be installed
  * `<targetURL>`: Defines the URL to the AEM instance to deploy to

* Example ui.apps POM file
```xml
<plugin>
    <groupId>com.day.jcr.vault</groupId>
    <artifactId>content-package-maven-plugin</artifactId>
    <extensions>true</extensions>
    <configuration>
        <group>aem-guides/wknd</group>
        <filterSource>src/main/content/META-INF/vault/filter.xml</filterSource>
        <embeddeds>
            <embedded>
                <groupId>com.adobe.aem.guides</groupId>
                <artifactId>wknd-sites-guide.core</artifactId>
                <target>/apps/wknd/install</target>
            </embedded>
        </embeddeds>
        <targetURL>http://${crx.host}:${crx.port}/crx/packmgr/service.jsp</targetURL>
    </configuration>
</plugin>
```

### Including Core Components
* Core Components: Set of base components used to speed up AEM development
  * Free and opensource on Github
* Installed automatically in the default runmode
  * In production runmode (nosamplecontent), you need to install them yourself by adding them to your Maven project
* 
