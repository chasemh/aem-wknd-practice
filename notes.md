# WKND Tutorial Site Notes

# Table of Contents
1. [Part 1: Project Setup](#part-1-project-setup)
2. [Part 2: Creating a Base Page and Template](#part-2-creating-a-base-page-and-template)
3. [Part 3: Client-Side Libraries and Responsive Grid](#part-3-client-side-libraries-and-responsive-grid)

* Tutorial found [here](https://helpx.adobe.com/experience-manager/kt/sites/using/getting-started-wknd-tutorial-develop.html)

## Part 1: Project Setup

* [Tutorial Part 1](https://helpx.adobe.com/experience-manager/kt/sites/using/getting-started-wknd-tutorial-develop/part1.html)
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

### Developer Workflow

* Devs push and pull code between local file system and local AEM instance
* Once code is locally tested, it should be pushed up to a shared repo, where it can be integrated and deployed.
* Something are easier to edit in AEM itself (Page layouts etc.). Edits can be made in AEM and then synced down into Eclipse.

## Part 2: Creating a Base Page and Template

* [Tutorial Part 2](https://helpx.adobe.com/experience-manager/kt/sites/using/getting-started-wknd-tutorial-develop/part2.html)

### Create Base Page Component

* Base page is responsible for ensuring global areas of the site are consistent
* Leads to loading of global CSS and JS and inclusion of code that will ensure a page can be edited via AEM authoring tools.
* Name `structure` matches any specific mode of Editable template
  * Any component under the `stucture` folder is a component meant to be used when building templates an not when authoring a page
* Templates are generic page layouts, pages are specific instances of templates
* Dialogs are the mechanism that allows authors to update properties and logic of component through a UI dialog box.
* HTML files in an AEM project are actually HTL files
* HTL files have a set of global objects that they can access (such as `currentPage` )

### Create Proxy Components for Text, Image, and Title

* AEM contains core components that can be used as basic building blocks for content creation
  * Text, Image, Title components
* Can include these in the WKND project by creating new cq:components for each one and setting them to inherit functionality from the core components via the `sling:resourceSuperType` setting
  * This is called creating proxy components
  * Recommended approach for including core components in your project
* cq:editConfig defines behavior such as Drag and Drop functionality from the Asset Finder in the Sites Editor

### Create Empty Template Type

* Template Types = Templates for templates. Need to take advantage of AEM's Editable Template feature
  * Stored under /conf in an AEM project
  * Typically edited within AEM directly
* Three main areas of Editable Templates
  1. Structure: Defines the components that are part of the template. Not editable by content authors.
  2. Initial Content: Defines components that the template will start with. Can be edited by content authors.
  3. Policies: Defines configurations on how components will behave and what options content authors have available.

### Creating Content Root

* Content root defines the allowed templates for a given site and is used to define other global configuration.
  * Conventionally not intended to be the home page for a site, should redirect to the true home page
* Once created, the content root can be added to source control as it is critical to the behavior of a site and provides a baseline content structure
* If the content root is very large, it is sometimes given it's own Maven module (ui.content)

## Part 3: Client-Side Libraries and Responsive Grid

* [Tutorial Part 3](https://helpx.adobe.com/experience-manager/kt/sites/using/getting-started-wknd-tutorial-develop/part3.html)

### Front End Frameworks
* Many third party frameworks can be easily integrated into AEM
* Example Frameworks
  * LESS: CSS pre-compiler that allows variables and other functionality in CSS. AEM client libs natively support LESS compilation.
  * Bootstrap: JS framework for site building
  * jQuery: JS library for manipulating HTML

### Clientlibs Structure
* Client-Side Libraries provides a mechanism to organize CSS, JS files needed for an AEM site implementation
* Basic Goals for client libraries (clientlibs)
  1. Store CSS/JS in small files for easier dev and maintenance
  2. Manage dependencies on third party frameworks in an organized fashion
  3. Minimize number of client-side requests by concatenating CSS/JS into one or two requests
  4. Minimize CSS/JS that is delivered to optimize speed/performance of a site.
* Conventional Layout of Clientlibs
  * clientlib-base: Embeds site specific and other clientlibs
  * clientlib-dependencies: Embeds third party dependency
  * clientlib-site: Site specific theme and style
  * vendor: Third party frameworks (Bootstrap, jQuery etc.)
  * clientlib-author: CSS/JS necessary for author environment only
* CSS is typically loaded at the top of a page
* JS is typically loaded at the bottom of page

### Implementing Client Libraries
* Client Library is just like another other type of AEM node
* Basic Structure  
  * jcr:primaryType: cq:ClientLibraryFolder
  * categories (String[])
  * embed (String[])
  * dependencies (String[])
  * css.txt (nt::file)
  * js.txt (nt::file)
* CSS and JS is dynamically loaded from a path that starts with `/etc.clientlibs`
* When a site is published, you should separate your client libraries away from `/apps` as you will want to restrict the path for security reasons.
* Can use `cq:htmlTag` to control the class attributes of certain HTML elements
  * There are issues syncing `cq:htmlTag` components from Eclipse to AEM
  * Make them in CRXDE Lite for now

### Configure Mobile Emulator

* Can setup mobile emulation of sites by creating a `sling:OsgiConfig` component based on the factory component  `com.day.cq.wcm.mobile.core.impl.MobileEmulatorProvider`
* The name must have a unique suffix added to the end of the factor name
  * i.e. `com.day.cq.wcm.mobile.core.impl.MobileEmulatorProvider-wknd`
* Typically place the component in a separate author config folder (`config.author`) to designate that authors should have the ability to work with the mobile emulator to preview pages on different types of mobile devices

### Front-End Developer Workflow

* Typically, some front end design work is done outside of AEM before implementation
* Static markup generation outside of AEM benefits
  * Determine and organize the front-end frameworks you want to use
  * Quicker to determine the global theme and style for the HTML
  * Discover issues between design and implementation early
  * Provides a reference for AEM developers
