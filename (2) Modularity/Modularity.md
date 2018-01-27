# Mastering Modularity

### Contents of This Tutorial
```
- What and Why Modularity
- Using metadata to describe OSGi bundles (aka modules)
- Explaining how bundle metadata is used to manage code visibility
- Illustrating how bundles are used to create an application
```

These notes will mostly focus on the purpose of the `Module` layer as it is one of the most attractive parts of OSGi. The `module` layer is the foundation on which everything else rests in the world of OSGi.

## What Even Is Modularity?

`Designing a complete system from a set of logically independent pieces (aka modules)`.

A module, more or less, enforces some type of logical boundary: code that is either part of a module (it's on the inside) or code that isn't part of a module (it's on the outside). The internal (actual implementation) of the module are visible to code that is `only` part of that specific module. For the other parts of the code, the only visible details of a module are those explicitly exposed (think of a public API of sorts).

## But What Is The Different Between Modularity and Standard OO?

Both of these principles are similar in the sense that they both care about `separation of concerns`. Ideally, you want to break down your system into minimally overlapping functionality or concerns so that each concern can be independently reasoned about, designed, implemented and used.

#### So... Why Do I Need Modularity if Object Orientation Handles My Separation of Concerns?

Let's visualize the construction of a new application that we are building (A notification widget). Thinking in terms of a UML architecture, we may have a `Notification` class. This notification obviously isn't going to do everything (Find the associated Person we are dealing with, connect to the SMTP server to enable a form of sending messages to a user and, obviously, handling the actual sending of the message.) No, this logic will be broken up..

- Notification Class
- Emailer Class
- SMTP Class
- Person Class

Something of the sort, as you can see, modularizes the widget into some compartmentalized functionality as to avoid tightly coupling a **it ton of dependencies and allowing for code reuse.

### The Problem With The Above Approach

Looking at the above approach, the dependencies exist within the inner-details of the code itself. Sure the notifier class will depend on some sort of email instantiation. Within that emailer class it may establish some connection using an SMTP connection class. The email class may also need to grab a reference to the actual person we are sending notifications too. As you see, there is a twisted route of class dependencies to ensure full functionality is present.

The problem is, this dependency isn't outlined anywhere but within the inner-workings of the code itself (IE CONFUSING...).

### What Is a Module?

```
- Set of logically encapsulated implementation classes

- An optional public API based on a subset of the implementation classes

- Set of dependencies on external code
```

### Why and How To Modularize

Modularity, as previously stated, focuses on separation of concerns to establish the following principles:

- *Cohesion* : Measures how closely aligned a module's classes are with each other. You shouldn't have a single module handle a bunch of different tasks (XML parsing, networking, persistence).

- *Coupling* : How tightly bound or dependent different modules are on each other. You want ***low*** coupling. You don't want every module to depend on all other modules.


From the book (OSGi In Action), we will be using the following paint example to express modularity in OSGi.

[***Click To View Paint Example Image***](https://www.safaribooksonline.com/library/view/osgi-in-action/9781933988917/02fig04_alt.jpg)

With this example above, we can drag and paint multiple shapes, move shapes around and change different shapes on the canvas.

Below represents the structure of an example `paint program` that is already coded. This program is packages as a single JAR file.

##### Directory Structure
```
META-INF/
META-INF/MANIFEST.MF
org/
org/foo/
org/foo/paint/
org/foo/paint/PaintFrame$1$1.class
org/foo/paint/PaintFrame$1.class
org/foo/paint/PaintFrame$ShapeActionListener.class
org/foo/paint/PaintFrame.class
org/foo/paint/SimpleShape.class
org/foo/paint/ShapeComponent.class
org/foo/shape/
org/foo/shape/Circle.class
org/foo/shape/circle.png
org/foo/shape/Square.class
org/foo/shape/square.png
org/foo/shape/Triangle.class
org/foo/shape/triangle.png
```

##### Descriptions

```
Class / Description
org.foo.paint.PaintFrame 	The main window of the paint program, which contains the toolbar and drawing canvas. It also has a static main() method to launch the program.
org.foo.paint.SimpleShape 	An interface representing an abstract shape for painting.
org.foo.paint.ShapeComponent 	A GUI component responsible for drawing shapes onto the drawing canvas.
org.foo.shape.Circle 	An implementation of SimpleShape for drawing circles.
org.foo.shape.Square 	An implementation of SimpleShape for drawing squares.
org.foo.shape.Triangle 	An implementation of SimpleShape for drawing triangles.
```

[Class Relationships Image](https://www.safaribooksonline.com/library/view/osgi-in-action/9781933988917/02fig05.jpg)

Currently, as shown above, everything is packaged into a single JAR file. This `single JAR file` already should be a red-flag showing that the program isn't `yet` modularized.

### Separation API's

It's good practice in OSGi to separate public API's into different packages so they can be easily shared without worrying about exposing implementation details.

The Paint Program provides a good example of a public API in the `SimpleShape` interface. The interface makes it easy to implement new, possibly third-party shapes for use with the program.

Unfortunately, `SimpleShape` is in the same package as the program's implementation classes.

In order to remedy this issue, you will move the `Simple-Shape` into the `org.foo.shape` package and move the implementations into the `org.foo.shape.impl` package.

These changes successfully divide the paint program into 3 logical pieces.

1) `org.foo.shape` - Public API for creating shapes.

2) `org.foo.shape.impl` - Various shape implementations

3) `org.foo.paint` - The application implementation

With the above (logical modularity), you could package each of these packages into a separate JAR file (physical modularity). To have OSGi verify and enforce the modularity, it isn't sufficient to package the code as JAR file, you must package them as bundles. To do this, you need to understand OSGI's `bundle concept` which is logical and physical units of modularity.

### Introducing Bundles

`Bundle` is how OSGi refers to its specific realization of the module concept. `Module` and `Bundle` can be used interchangably.

##### Bundle

`
A physical unit of modularity in the form of a JAR file containing code, resources and metadata. The boundary of the JAR file also serves as the encapsulation boundary for logical modularity at execution time.
`
Ultimately, the only differences from a JAR file and a Bundle JAR file is the module metadata, which is used by the OSGi framework to manage its modularity characteristics.

##### Bundle Metadata
All metadata belongs in the `META-INF/MANIFEST.MF` entry of the JAR file. This is where OSGi places its module metadata.

### Quickly Differentiation Physical & Logical Modularity

Physical: How code is packaged and/or made available for deployment

Logical: The form of code visibility you define in your application

### Bundle Role in Physical Modularity

A bundle with respect to `physical modularity` refers to the module membership. No metadata is required to associate making a class a member of a bundle.

A given class is a member of a bundle if it's contained in the bundle JAR file. Added benefit? You don't need to actually do anything special to make a class a member of a bundle, you just put it in the bundle JAR file.

You can then deploy these bundles are tangible, singular artifacts to the application server or environment.


### Where Does The Metadata Go?

This is a controversial topic. My opinion on this topic leans toward the ability to keep metadata in a separate file alongside the source code instead of baking the metadata into the source code itself. Why?

- Don't need to recompile your bundle to make changes to the metadata
- Don't need access to the source code to add or modify metadata, which is sometimes necessary when dealing with legacy or third-party libraries.
- You can run your code on older or smaller JVM's that don't support annotations

### Bundle Role in Logical Modularity

The bundle role in logical modularity is to logically encapsulate member classes.

Hey, what does that mean? Well, this specifically relates to code visibility.

(EXAMPLE): Say you have a utility class in a util package that isn't part of your project's public API. To use this utility class from a different package, it must be public. Well.. if it's public then anyone can use it, even if it's not a part of the public API.

The `logical` boundary that is created by a bundle changes this, giving classes inside the bundle different visibility rules to external code. This means that public classes inside your bundle JAR file aren't necessarily externally visible.

ONLY code explicitly exposed via bundle metadata is visible externally. You can think of this logical boundary effectively extending the idea behind the standard `public, private and protected` capabilities in the Java ecosystem to module visiblity (only visible within the module).

### How Do We Use Metadata to Describe Bundles?

Metadata more or less exists for the OSGi framework to handle resolving dependencies and enforcing encapsulation.

- Human readable information - optional information intended to aid humans to understand the bundle they are using
- Bundle identification - Required information to identify a bundle
- Code visibility - Required information for defining which code is internally visible and which internal code is externally visible

The syntax for the manifest file follows the structure:
```
name: value
```

Example Manifest File

```
Manifest-Version: 1.0
Created-By: 1.4 (Sun Microsystems Inc.)
Bundle-ManifestVersion: 2
Bundle-SymbolicName: org.foo.api
Bundle-Version: 1.0.0.SNAPSHOT
Bundle-Name: Simple Paint API
Export-Package: org.foo.api
Import-Package: javax.swing,org.foo.api
Bundle-License: http://www.opensource.org/licenses/apache2.0.php
```

OSGi manifest syntax may look like the following:
```
Property-Name: clause, clause, clause
```

#### Human Readable Metadata

Some metadata solely exists to `explain` things to the developer utilizing the bundle.

```
Bundle-Name: Simple Paint API
Bundle-Description: Public API for a simple paint program.
Bundle-DocURL: http://www.manning.com/osgi-in-action/
Bundle-Category: example, library
Bundle-Vendor: OSGi in Action
Bundle-ContactAddress: 1234 Main Street, USA
Bundle-Copyright: OSGi in Action
```

Looking back at the first manifest file, we must disect what the following attributes do for the OSGi framework.

- Bundle-Name: Purely for the user to understand what the bundle is doing
- Bundle-SymbolicName: This is how the OSGi framework actually locates and references a particular bundle

You can use the package as the symbolic name since it already follows the reverse-domain scheme dot separated syntax.

Because versions change and continuously changing the bundle name would be more and more cumbersome, the Bundle-Version adds an additional layer of identification to the bundle. This value conforms to the `OSGi version number format`.

```
Bundle-SymbolicName: org.foo.shape
Bundle-Version: 2.0.0
```

`Bundle-ManifestVersion: 2` is necessary if you need to maintain backward compatibility with legacy bundles before OSGi R4.

### Code Visibility

The OSGi spec allows for metadata to describe which code is visible internally in a bundle and which internal code is visible externally.

- Internal Bundle Class Path: The code forming the bundle
- Exported Internal Code: Explicitly exposed code from the bundle class path for sharing with other bundles
- Imported External Code: External code on which the bundle class path code depends

In a standard JAR file, the Java class path will search for all directories that exist within the root class path of the JAR until a given class is found.

On the contrary, for OSGi, there is the notion of a bundle class path. This is an ordered, comma-separated list to tell other bundle jar locations where to search explicitly for a given class.

Classes in the same bundle have access to all code reachable on their bundle class path.

The bundle class path is defined in the MANIFEST using the following attribute name: `Bundle-ClassPath` which will have associated (comma-separated values) of the internal classes within the bundle and their respective locations.

`Bundle-ClassPath: .,other-classes/,embedded.jar`

Something like the above will tell the classloader where to look for different classes or resources.

The `.` above signifies the bundle JAR file. This shows that the bundle is searched first for root-relative packages.

### Extremely Good Explanation of Bundle-ClassPath and Import-Packages

[CLICK HERE TO READ](https://stackoverflow.com/a/16939053/8891397)

### Flexibility of the Bundle Class Path
The bundle class path doesn't apply only to classes, but also to external resources as well.

For example, you may want to package an external XML resource, .png image or html file into a resources directory.

In some situations, you may have a legacy JAR file that cannot be changed. In this case you could utilize an `embedded JAR` approach, you can add the JAR into the metadata and use it without changing any of the source code

`
Another benefit of embedding a jar is that if you want a private copy of a JAR file without having to share the same static member variables utilized by other Classes.
`

### Class In A Bundle Visibility Explained

A standard JAR exposes everything relative to the root by default. Within OSGi, NOTHING is exposed by default.

To take advantage of exposing externally useful code, such as an API for your clients, then must take use of the `Export-Package` syntax in the `MANIFEST`.

The value for this attribute is a comma separated list of packages to share with other bundles.

##### Important Note About Sharing
OSGi `doesn't` share at the class-level, but at the `package level`.

```
Export-Package: org.foo.shape; vendor="Manning", org.foo.other;
 vendor="Manning"
 ```

 Attaching a vendor attribute allows you to distinguish who is exporting what if multiple bundles are exporting the same package.

##### Summary Note:
 `
 Both Bundle-ClassPath and Export-Package deal with the visibility of internal bundle code. Normally, a bundle is also dependent on external code.
 `

 [CLICK HERE TO READ BUNDLE-CLASSPATH/IMPORT TRADE_OFF](https://stackoverflow.com/a/16939053/8891397)


 ### Understanding Importing External Packages

 There is a high chance that your bundle classes will require access to external code outside of the bundle you are dealing with. In order to deal with this, you can use the `Import-Package` syntax. Not to be confused with the standard Java `import` keyword for namespace resolution, `Import-Package` specifically allows your bundle to import external dependencies explicitly.

 One thing to note, external packages that are included will not automatically include nested packages of those included packages. Those need to be included as well.

 ```
 Import-Package: org.foo.shape; vendor="Manning"
 ```

 ### Summary

 We’ve covered a lot of ground in this chapter. Some of the highlights include the following:

```
    - Modularity is a form of separation of concerns that provides both logical and physical encapsulation of classes.

    - Modularity is desirable because it allows you to break applications into logically independent pieces that can be independently changed and reasoned about.
    Bundle is the name for a module in OSGi. It’s a JAR file containing code, resources, and modularity metadata.

    - Modularity metadata details human-readable information, bundle identification, and code visibility.

    - Bundle code visibility is composed of an internal class path, exported packages, and imported packages, which differs significantly from the global type assumption of standard JAR files.

    - The OSGi framework uses the metadata about imported and exported packages to automatically resolve bundle dependencies and ensure type consistency before a bundle can be used.

    - Imported and exported packages capture inter-bundle package dependencies, but uses constraints are necessary to capture intra-bundle package dependencies to ensure complete type consistency.

```
