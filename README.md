# Micronaut Demo

A demonstration project showcasing core Micronaut framework features in a modular plugin architecture.

## Overview

This project demonstrates a plugin-based application engine built with Micronaut, highlighting:

- **Dependency Injection (DI)** using Jakarta annotations
- **Event-driven architecture** with Micronaut's event system
- **Multi-module architecture** with clear separation of concerns
- **Compile-time annotation processing** for zero reflection overhead

## Architecture

### Modules

- **engine-api** - Defines plugin and event interfaces
- **engine** - Core engine with plugin management and lifecycle
- **plugin** - Example plugin implementation
- **application** - Main application entry point
- **engine-bundle** - Aggregated distribution bundle

### Key Components

```
┌─────────────┐
│ Application │ implements IApplication
└──────┬──────┘
       │
       ▼
   ┌────────┐
   │ Engine │ manages lifecycle
   └───┬────┘
       │
       ▼
┌──────────────┐
│PluginManager │ discovers & manages plugins
└──────┬───────┘
       │
       ▼
  ┌────────┐
  │ Plugin │ implements IPlugin
  └────────┘
```

## Micronaut Features

### Dependency Injection

All components use `@Singleton` for automatic bean registration and constructor injection:

```java
@Singleton
@RequiredArgsConstructor
public class Engine {
  private final IApplication application;
  private final PluginManager pluginManager;
  private final ApplicationEventPublisher<Ping> publisher;
  // ...
}
```

**Key DI Features:**
- Constructor injection with Lombok's `@RequiredArgsConstructor`
- Interface-based injection (`IApplication`, `IPlugin`)
- Automatic collection injection (`List<IPlugin>` in `PluginManager`)
- Cross-module dependency injection (application injects engine and plugin beans)

### Event System

The project demonstrates Micronaut's event publishing and listening:

**Publishing Events:**
```java
@Singleton
public class Engine {
  private final ApplicationEventPublisher<Ping> publisher;
  
  public void start() {
    publisher.publishEvent(new Ping("ping 1"));
  }
}
```

**Listening to Events:**
```java
@Singleton
public class ExamplePlugin implements IPlugin {
  @EventListener
  public void onPing(Ping ping) {
    log.info("RECEIVED PING!: {}", ping.message());
  }
}
```

**Event Types:**
- `Ping` - Simple message events
- `PluginStarted` / `PluginStopped` - Plugin lifecycle events
- `EngineStarted` / `EngineStopped` - Engine lifecycle events

Events are published synchronously and received by all registered listeners across modules.

### Plugin Discovery

The `PluginManager` automatically discovers all `IPlugin` implementations through DI:

```java
@Singleton
@RequiredArgsConstructor
public class PluginManager {
  final List<IPlugin> plugins;  // Auto-injected by Micronaut
  
  public void start() {
    plugins.forEach(p -> {
      p.start();
      pluginStartedApplicationEventPublisher.publishEvent(new PluginStarted(p));
    });
  }
}
```

Micronaut's compile-time annotation processor discovers all `@Singleton` beans implementing `IPlugin` and injects them as a list.

## Building & Running

### Requirements
- Java 21
- Maven 3.6+

### Build
```bash
mvn clean package
```

### Run
```bash
cd application
java -jar target/application-1.0-SNAPSHOT.jar
```

### Run Tests
```bash
mvn test
```

## Example Output

When running the application, you'll see the event-driven lifecycle in action:

```
INFO  engine.Engine - Starting engine...
INFO  plugin.ExamplePlugin - Starting plugin...
INFO  engine.EngineListener - plugin started: ExamplePlugin
INFO  application.DemoEventListeners - Received ping event: ping 1
INFO  plugin.ExamplePlugin - RECEIVED PING!: ping 1
INFO  application.Main - Starting application...
INFO  application.DemoEventListeners - Received ping event: ping 2
...
```

## Testing

The project includes both unit and integration tests:

- **Unit Tests** - Simple tests without DI context (e.g., `MainTest`)
- **Integration Tests** - Tests with full Micronaut context using `@MicronautTest`:

```java
@MicronautTest
class MainIT {
    @Inject
    private Engine engine;
    
    @Inject
    private ExamplePlugin examplePlugin;
    
    @Test
    void testContextStartsAndBeansAreInjected() {
        assertThat(engine).isNotNull();
        assertThat(examplePlugin).isNotNull();
    }
}
```

## Key Takeaways

1. **Zero-reflection DI**: Micronaut processes annotations at compile-time for fast startup
2. **Type-safe events**: Events are strongly typed with automatic listener discovery
3. **Modular by design**: Clean separation between API, implementation, and plugins
4. **Cross-module injection**: Beans from different modules are seamlessly wired together
5. **Minimal configuration**: Convention-over-configuration approach with sensible defaults
