+++ 
draft = false
date = 2025-11-03
title = "Understanding Proxies in Java: Static vs Dynamic Explained Simply"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

###  Understanding Proxies in Java: Static vs Dynamic Explained Simply


##  Introduction

Have you ever used Spring’s @Transactional, @Cacheable, or even a Hibernate entity that loads “just in time”?
Guess what you’ve already been using proxies without even realizing it.

Proxies are everywhere in modern Java silently adding features like logging, caching, or security checks all without touching your original code.
They’re one of those “behind the curtain” concepts that make big frameworks magical.

Let’s pull that curtain back a little.

---

##  What is a Proxy?
A proxy is a wrapper around a real object, think of a **proxy** as a middleman.  
When you call a method on an object, instead of directly calling it, you talk to a **proxy**, and the proxy talks to the real object for you.

It’s like having a **manager** who answers calls for a celebrity —  
they screen the calls, maybe record them, and then pass them along if appropriate.

So in Java:
- The **real object** = the celebrity  
- The **proxy** = the manager  
- The **caller (you)** = the audience  

A proxy can:
- Log information before and after a method call  
- Check permissions  
- Manage transactions  
- Add caching or retry logic  

So, a proxy isn’t replacing your real object it’s just *standing in front of it*, adding some extra behavior.

---
##  Types of Proxies in Java
There are two main types of proxies in Java:
1. **Static Proxies**: You write the proxy class yourself.

2. **Dynamic Proxies**: Java creates it for you at runtime using reflection. 

---
###  Static Proxies
### What are Static Proxies?
A **static proxy** is the simplest kind of proxy one you **write manually**.

You literally create another class (the proxy) that implements the same interface as your real object.  
This proxy class wraps the **real object**, and inside each method, it adds **extra behavior** like logging, validation, or timing — before or after calling the real method.

Think of it like a **personal assistant**:
> Before you talk to the boss (real object), the assistant (proxy) takes your message, checks it, maybe notes the time, and then passes it along.

### Example — Without Proxy

Let’s start with a normal example without any proxy:

```java
public interface Service {
    void performTask();
}

public class RealService implements Service {
    @Override
    public void performTask() {
        System.out.println("Performing the actual task");
    }
}

public class Main {
    public static void main(String[] args) {
        Service service = new RealService();
        service.performTask();
    }
}
```
Output:
```
Performing the actual task
```
Simple and clean — but what if we want to log every time this task runs?
We could modify RealService, but that mixes business logic with logging logic — not good design.

### Implementing a Static Proxy

Instead of modifying RealService, let’s create a proxy class that wraps it.

Step 1: Implement the same interface
 - The proxy must look like the real service so it implements the same interface.

```java
public class ServiceProxy implements Service {
    private final Service realService;

    public ServiceProxy(Service realService) {
        this.realService = realService
    };
    override
    public void performAction() {   
        System.out.println("Proxy Logging before task);
        realService.performAction();
        System.out.println("Proxy Logging before task");
    }
}
```    
Step 2: Use it in the client code
```java
public class Main {
    public static void main(String[] args) {
        Service realService = new RealService();
        Service proxy = new ServiceProxy(realService); 
        proxy.performAction();
    }
}
```
Output: 
```
Proxy Logging before task
Performing the actual task
Proxy Logging after task
```
---
### What Happens Behind the Scenes?
When you call proxy.performTask(), this happens:
1. The call goes to the ServiceProxy’s performTask() method.
2. The proxy logs “Proxy Logging before task”.
3. It then calls the realService.performTask() method.
4. The real service executes its logic and prints “Performing the actual task”.
5. After that, the proxy logs “Proxy Logging after task”.   

So your client talks to the proxy, but it feels like talking to the real object.

### Advantages of Static Proxies
- Simple to understand and implement.
- You have full control over the proxy behavior.
- No reflection or complex mechanisms involved.

### Limitations of Static Proxies

- **Boilerplate Code**: You have to write a new proxy class for every interface you want to proxy.
- **Tight Coupling**: The proxy is tightly coupled to the real object, making it less flexible.
- **Maintenance Overhead**: If the interface changes, you have to update both the real class and the proxy class.

---
### Visual Representation

Here’s a simple flow of how a static proxy works:
```
[Client]
   ↓
calls method on
   ↓
[Proxy Object]
   ↳ Logs info, validates data, or checks security
   ↓
delegates to
   ↓
[Real Object]
   ↳ Executes actual business logic
```
---

### Dynamic Proxies
Static proxies are easy to understand, but they become painful when your project grows.
Imagine you have **10 different interfaces** — writing 10 different proxy classes just to add logging or security checks would be exhausting.
Wouldn’t it be nice if Java could **generate those proxy classes automatically** for you?
That’s exactly what **Dynamic Proxy** does.

---
### What are Dynamic Proxies?
A **dynamic proxy** is created at runtime using Java’s reflection capabilities.
Dynamic proxies solve the “too many proxy classes” problem.
You just tell Java:
“Hey, here’s the interface and the real object. Whenever someone calls a method, send it to my handler first.”
Java will generate the proxy class in memory, handle all method calls, and delegate to your real object when needed.

With dynamic proxies, you create a single **invocation handler** that defines the extra behavior you want (like logging or security checks).
Then, at runtime, Java generates a proxy class that implements the desired interface and routes method calls through your invocation handler.

---
### Here’s how it works step by step:

Step 1: Create an interface
```java
public interface Service {
    void performAction();
}
```
Step 2: Implement the Real Object
```java
public class RealService implements Service {
    @Override
    public void performAction() {
        System.out.println("Doing the real work in RealService");   
    }
}
```
Step 3: Create an InvocationHandler

Here’s the heart of the dynamic proxy system the InvocationHandler.
It defines what happens when a method is called on the proxy.

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;    

public class LoggingHandler implements InvocationHandler {
    private final Object realObject;

    public LoggingHandler(Object realObject) {
        this.realObject = realObject;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Proxy logging before task: " + method.getName());
         // call the actual method on the real object
        Object result = method.invoke(realObject, args);
        
        System.out.println("Proxy logging after task" + method.getName());
        return result;
    }
}
```
Step 4: Create the Dynamic Proxy
Now, we use Java’s Proxy class to create a dynamic proxy instance.
```java
import java.lang.reflect.Proxy; 
public class Main {
    public static void main(String[] args) {
        Service realService = new RealService();
        
        Service proxyInstance = (Service) Proxy.newProxyInstance(
            Service.class.getClassLoader(),
            new Class[]{Service.class},
            new LoggingHandler(realService)
        );

        proxyInstance.performAction();
    }
}
```
Output:
```
Proxy logging before task: performAction
Doing the real work in RealService
Proxy logging after task: performAction
```
---
### What Happens Behind the Scenes?
When you call proxyInstance.performAction(), this happens:

1. The call goes to the dynamic proxy instance created by `Proxy.newProxyInstance()`.
2. The proxy routes the call to the `invoke()` method of `LoggingHandler`.
3. Inside `invoke()`, it logs “Proxy logging before task: performAction”.
4. It then calls the actual method on the real object using `method.invoke(realObject, args)`.
5. The real service executes its logic and prints “Doing the real work in RealService”.
6. After that, the proxy logs “Proxy logging after task: performAction”.  

So your client talks to the dynamic proxy, but it feels like talking to the real object.

---
###  Key Classes Involved

| **Class / Interface** | **Role** |
|------------------------|----------|
| `java.lang.reflect.Proxy` | Used to create proxy objects dynamically at runtime. |
| `java.lang.reflect.InvocationHandler` | Interface you implement to define what happens when proxy methods are invoked. |
| `invoke(Object proxy, Method method, Object[] args)` | Called every time a method is invoked on the proxy. |

---
### How Does Java Create Dynamic Proxies?
When you call `Proxy.newProxyInstance()`, Java does the following:
1. It generates a new class in memory( like $Proxy0) that implements your interfaces.
2. This generated class  doesn’t contain real logic — instead, routes all method calls to the `invoke()` method of your `InvocationHandler`.
3. The generated proxy class is loaded by the JVM and used like any other Java object.

You can even inspect these proxy classes if you enable debugging flags:
```java
System.getProperties().put("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");
```
    This will save the generated proxy class files to your working directory, allowing you to see how Java creates them.

---
### Advantages of Dynamic Proxies
- **Less Boilerplate**: You only need one handler class for multiple interfaces.
- **Flexibility**: You can change the behavior of the proxy at runtime by changing the handler logic.
- **Decoupling**: Your real objects remain clean and focused on business logic.

### Limitations of Dynamic Proxies
- **Interface Requirement**: Dynamic proxies only work with interfaces. If you have concrete classes, you’ll need to use other techniques (like CGLIB).
- **Performance Overhead**: There’s a small performance hit due to reflection, but it’s usually negligible for most applications.
- **Complexity**: Understanding dynamic proxies and reflection can be tricky for beginners. 

---
### Visual Representation
Here’s a simple flow of how a dynamic proxy works:
```[Client]
   ↓
calls method on
   ↓
[Dynamic Proxy Object]
   ↳ Routes call to InvocationHandler
   ↓
[InvocationHandler]
   ↳ Logs info, validates data, or checks security
   ↓
delegates to
   ↓
[Real Object]
   ↳ Executes actual business logic
```

### Real World Use Cases of Proxies
- **Spring Framework**: Uses dynamic proxies extensively for features @Transactional, @Cacheable, @Async, etc.
- **Hibernate**: Uses proxies for lazy loading of entities
- **Security frameworks**: Wrap your methods to enforce authentication or authorization.
- **Logging frameworks**: Intercept calls to log inputs and outputs automatically.

---
###  Static vs Dynamic Proxy — Comparison Recap

| **Feature** | **Static Proxy** | **Dynamic Proxy** |
|--------------|------------------|-------------------|
| **Creation** | Manually written | Generated at runtime |
| **Reflection** | ❌ No | ✅ Yes |
| **Works with** | Classes or interfaces | Interfaces only |
| **Code size** | More boilerplate | Minimal |
| **Flexibility** | Low | High |
| **Common Use** | Simple apps | Frameworks (Spring, Hibernate) |

---
### When to Use What
- Use **Static Proxies** when:
  - You have a small number of interfaces.
  - You want full control over the proxy implementation.
  - Performance is critical, and you want to avoid reflection overhead.
- Use **Dynamic Proxies** when:
  - You have many interfaces to proxy.
  - You want to add cross cutting concerns (like logging, security) without cluttering your business logic.
  - You want to leverage frameworks that use dynamic proxies (like Spring). 

---
### In a Nutshell
- A proxy stands between the client and the real object.
- It adds behavior like logging, security, or transactions — without changing the real code.
- Static Proxy: You write it yourself.
- Dynamic Proxy: Java generates it at runtime.
- InvocationHandler: The brain behind dynamic proxies — decides what happens when a method is called.

---

### Final Thoughts

Once you understand proxies, you’ll start seeing them everywhere — in Spring, Hibernate, REST clients, even security filters.
They’re the quiet guardians of your code — the ones doing all the behind-the-scenes work while you focus on writing clean, simple logic.
So next time something “magically” happens in your Spring app, remember —
there’s probably a proxy making it happen.
