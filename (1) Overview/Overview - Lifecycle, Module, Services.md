# Let's Learn OSGi


### What is OSGI?

OSGI is known as a "modularity layer" for the Java platform.

### What exactly do you mean by modularity platform?

The code of your application is divided up more or less into a series of logical parts which represent different functional concerns of your application. Having modular software with help simplify the development process as well as improve code maintainability.

### OSGI Architecture

OSGI is essentially broken up into 2 components.
  - Standard Services: Defines reusable APIs for common tasks such as logging and preferences.
  - OSGi Framework: The runtime that implements and provides OSGi functionality.

More for this can be found @ http://osgi.org/

#### The 3 Layers of the OSGi Architecture

- Service Layer
- Lifecycle Layer
- Module Layer

##### Service Layer
Concerned with the interaction and communication among modules and the components contained within the modules

#####  Lifecycle Layer
Concerned with providing execution module management and access to underlying OSGi framework

##### Module Layer
Concerned with packaging and sharing code

Each layer listed above is dependent on the layers beneath it.

### Understanding The Module Layer

The module ideology forms the creation of what is known as a *bundle*.  A bundle is a fancy name for a standard JAR that contains some additional metadata. This metadata can explicitly declare which packages are externally visible. In another sense,
this metadata is an additional feature to the standard Java access modifiers such as public, private and protected properties.

### Understanding The Lifecycle Layer

The purpose of the lifecycle layer is to do two things involving bundles being dynamically installed and managed in the OSGi framework.

*Analogy: If you were building a house, the module layer provides the foundation and structure while the lifecycle layer would be the electric wiring*.

The two purposes that this layer provides is centric upon:
 - Defines the bundle lifecycle operations:
  - install
  - update
  - start
  - stop
  - uninstall

With these features, the lifecycle operations allow you to dynamically administer, manage and evolve your application. In simpler terms, you can (albeit, very easily) install and uninstall bundles from your application without restarting the entire application process.

### Understanding The Service Layer

The service layer focuses on the *publish, find and bind* interaction pattern. Services providers will publish their services into a service registry. Service clients will search the registry to find available services to user.

This layer is considered good practice because it separates the interface from the implementation.

#### But What Exactly Is A Service?

A service is nothing fancy, it's simply a Java interface which represents a conceptual contract between service providers and service clients.

### Putting It ALL Together

You may be wondering, how exactly are all these layers used together?

1. You want to design your application by breaking it down into service interfaces (normal interface-based programming) and clients who are going to use and implement those interfaces.

2. Implement the service provider and client components.

3. Package your service provider and client components into separate JAR files.

4. Start the OSGi Framework

5. Install and start all your component JAR files.

If you typically use the standard interface approach, this may feel very similar. On the flip-side of things, this differs in that pattern because the services will publish themselves in the service registry and the clients will look up available services in the registry.

### Module Layer Example (Hello World Impl)

```
package org.foo.hello;

public class Greeting {
  final String m_name;

  public Greeting(String name) {
    m_name = name;
  }

  public void sayHello() {
    System.out.println("Hello, " + m_name + "!");
  }
}
```

During the build process you will compile and generate a JAR for the aforementioned class. In order for you to actually use the OSGi module layer, you must add some *metadata* into the Jars `META-INF/MANIFEST.MF` file:

##### Example Manifest File (Export-Package Example)
```
Bundle-ManifestVersion: 2                       // Metadata syntax version
Bundle-Name: Greeting API                       // Human readable name (optional)
Bundle-SymbolicName: org.foo.hello              // Symbolic Name
Bundle-Version: 1.0                             // Bundle Version
Export-Package: org.foo.hello;version="1.0"     // Shares packages with other bundles
```

The bulk of the above metadata essentially is related to the identification of the bundle itself. On the flip-side, the `export-package` statement extends the functionality of a typical JAR file with the ability for you to explicitly declare which packages contained in the JAR are visible to its users.

With this knowledge, you can see in the above example that only the contents of `org.foo.hello` are externally visible. This is beneficial because other modules won't be able to accidentally depend on packages your module doesn't explicitly expose.

##### Example Manifest File (Import-Package Example)
```
Bundle-ManifestVersion: 2                       // Metadata syntax version
Bundle-Name: Greeting Client                    // Human readable name (optional)
Bundle-SymbolicName: org.foo.hello              // Symbolic Name
Bundle-Version: 1.0                             // Bundle Version
Import-Package: org.foo.hello;version="1.0"     // Shares packages with other bundles
```


### Lifecycle Layer Example (Hello World Impl)

The lifecycle method is a deeper approach that the simple `module` method. Example use-case is thinking of wanting to start a background thread upon the initialization of a piece of software, the lifecycle layer can aide in this process.

*Bundles* may declare a given class as an activator, which is the bundle's hook into its own lifecycle management.

[Extended Greeting Class Example](https://www.safaribooksonline.com/library/view/osgi-in-action/9781933988917/014fig01.jpg)

##### OSGi bundle activator for the greeting implementation
```
package org.foo.hello;

import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;

public class Activator implements BundleActivator {

  public void start(BundleContext ctx) {
    Greeting.instance = new Greeting("lifecycle");
  }

  public void stop(BundleContext ctx) {
    Greeting.instance = null;
  }
}
```

What do we get here? The client can now easily use the preconfigured singleton instead of having to worry about creating its own instance.

##### Bundle Activate Requirements

- Must implement a simple OSGi interface (BundleActivator) that contains two methods (start and stop)

The `BundleContext ctx` parameter in the `start()` and `stop()` method is what is used to give the bundle access to the OSGi framework in which it's executing. The `ctx` object gives the module access to the modularity, lifecycle and services. (This will be discussed in more detail later).

If you have an existing JAR file that you want to convert into a module, you must add the activator implementation to the existing project so the class is included in the resulting JAR file.

Additionally, you must tell the OSGi framework about the bundle activator by adding another piece of metadata to the manifest.

```
Bundle-Activator: org.foo.hello.Activator
Import-Package: org.osgi.framework
```

##### Lifecycle Summary:

This section showed how to create OSGi bundles out of existing JAR files using the module layer and how to make the bundle lifecycle away so they can use the framework functionality.


### Service Layer Example

Let's discuss interfaces...

```
package org.foo.hello;
public interface Greeting {
  void sayHello();
}
```

For any given implementation of the Greeting interface, if the `sayHello()` method is invoked, a greeting will be displayed.

A `service` represents a contract between a provider and prospective clients. The semantics of such a contract are typically organized in some sort of spec or external documentation for the software.

The interface defined in the code above just represents the syntatic contract of all `Greeting` implementations. This notion of a `contract` is necessary because clients can be assured of getting the functionality they expect when using the `Greeting Service`.

The underlying details of how the `Greetin` implementation performs its tasks aren't directly known to the client. For example:
- Client (A) could print the greeting textually
- Client (B) could display the greeting via some sort of GUI within a dialog window

*Textual Impl of the Greeting Interface*
```
package org.foo.hello.impl;

import org.foo.hello.Greeting;

public class GreetingImpl implements Greeting {
  final String m_name;

  GreetingImpl(String name) {
    m_name = name;
  }

  public void sayHello() {
    System.out.println("Hello, " + m_name + "!");
  }
}
```

You're on the right path if you're wondering `how is this even different than a standard interface/impl approach?` And you're absolutely right, NOTHING! The differences will appear in the following two ways:

1) How the service instance is available to the rest of the application

2) How the rest of the application is able to discover the aforementioned service

All service implementations are packaged into a bundle and that bundle must be lifecycle aware in order to register the service. (Think back to the service layer architecture)

```
Service Layer
Lifecycle Layer
Module Layer
```

But what does this actually `MEAN?`. Well, this means we need to create a bundle activator for the example service. Creating a bundle activator will enable the service to be `lifecycle aware`.

*Bundle Activator With Service Registration*

```
package org.foo.hello.impl;

import org.foo.hello.Greeting;
import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;

public class Activator implements BundleActivator {

  public void start(BundleContext ctx) {
    //Register service will register the service and create a new instance during the initialization of the service
    ctx.registerService(Greeting.class.getName(),
        new GreetingImpl("service"), null);
  }

  public void stop(BundleContext ctx) {}
}
```

##### How Do You Access A Service Reference In OSGi?

Accessing a Service Reference is a `TWO-STEP PROCESS`:
- Indirect reference is retrieved from the service registry

```
ServiceReference ref = ctx.getServiceReference(Greeting.class.getName());
```

- This indirect reference (above) is used to access the service object instance.

```
((Greeting) ctx.getService(ref)).sayHello();
```

<b>Philosophy</b>

The service implementation and the client should be packaged into separate bundle JAR files. The metadata for each bundle declares its corresponding activator, but the service implementation exports the `org.foo.hello` package, whereas the client imports it.


The client bundles metadata only needs to declare an import for the `Greeting` interface package. It has no direct dependency on the actual service implementation. This makes it super easy to swap service implementations dynamically without restarting the client bundle.


#### Principle Wrap-Up

- If you want only better modularity, use the module layer
- If you want to initialize modules and interact with the module layer, use the lifecycle layer.
- If you want a dynamic, interface-based development approach, use all three layers.
