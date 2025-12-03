---
name: maven-library-writer
description: Write Java libraries following best practices for Maven-based projects. Use when creating new Java libraries, refactoring existing libraries, designing library APIs, or when the user wants clean, minimal, production-ready Java library code. Triggers on requests like "create a library", "write a Java library", "design a library API", or mentions of Maven library patterns.
---

# Maven Library Writer

Write Java libraries following best practices from successful open-source projects and enterprise standards.

## Core Philosophy

**Simplicity over cleverness.** Minimal dependencies. Explicit code over reflection magic. Framework integration without tight coupling. Every pattern serves production use cases.

## Project Structure

Every Maven library follows this structure:

```
library-name/
├── pom.xml                           # Maven configuration
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/library/
│   │   │       ├── Library.java      # Main API entry point
│   │   │       ├── config/           # Configuration classes
│   │   │       ├── core/             # Core functionality
│   │   │       └── util/             # Utility classes
│   │   └── resources/
│   │       └── META-INF/
│   │           └── spring.factories  # (Optional) Spring Boot auto-config
│   └── test/
│       ├── java/
│       │   └── com/company/library/
│       └── resources/
└── README.md                         # Documentation
```

## POM.xml Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company</groupId>
    <artifactId>library-name</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>Library Name</name>
    <description>Clear, concise description of what the library does</description>
    <url>https://github.com/company/library-name</url>

    <licenses>
        <license>
            <name>MIT License</name>
            <url>https://opensource.org/licenses/MIT</url>
        </license>
    </licenses>

    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- Keep dependencies minimal -->
        <!-- Only include what's absolutely necessary -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>2.0.9</version>
        </dependency>

        <!-- Test dependencies -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

## Main Entry Point Pattern

```java
package com.company.library;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Main entry point for the Library API.
 * 
 * Example usage:
 * <pre>
 * Library library = Library.builder()
 *     .timeout(Duration.ofSeconds(30))
 *     .retryCount(3)
 *     .build();
 * 
 * Result result = library.process(input);
 * </pre>
 */
public class Library {
    private static final Logger logger = LoggerFactory.getLogger(Library.class);
    
    private final Duration timeout;
    private final int retryCount;
    private final boolean debugMode;
    
    private Library(Builder builder) {
        this.timeout = builder.timeout;
        this.retryCount = builder.retryCount;
        this.debugMode = builder.debugMode;
    }
    
    public static Builder builder() {
        return new Builder();
    }
    
    public Result process(Input input) {
        // Implementation
        return new Result();
    }
    
    public static class Builder {
        private Duration timeout = Duration.ofSeconds(30);
        private int retryCount = 3;
        private boolean debugMode = false;
        
        public Builder timeout(Duration timeout) {
            this.timeout = timeout;
            return this;
        }
        
        public Builder retryCount(int retryCount) {
            this.retryCount = retryCount;
            return this;
        }
        
        public Builder debugMode(boolean debugMode) {
            this.debugMode = debugMode;
            return this;
        }
        
        public Library build() {
            return new Library(this);
        }
    }
}
```

## Configuration Pattern

For Spring Boot integration (optional):

```java
package com.company.library.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(LibraryProperties.class)
public class LibraryAutoConfiguration {
    
    @Bean
    public Library library(LibraryProperties properties) {
        return Library.builder()
            .timeout(properties.getTimeout())
            .retryCount(properties.getRetryCount())
            .debugMode(properties.isDebugMode())
            .build();
    }
}

@ConfigurationProperties(prefix = "library")
public class LibraryProperties {
    private Duration timeout = Duration.ofSeconds(30);
    private int retryCount = 3;
    private boolean debugMode = false;
    
    // Getters and setters
}
```

## Exception Handling

```java
package com.company.library;

/**
 * Base exception for all library errors.
 */
public class LibraryException extends RuntimeException {
    public LibraryException(String message) {
        super(message);
    }
    
    public LibraryException(String message, Throwable cause) {
        super(message, cause);
    }
}

/**
 * Thrown when configuration is invalid.
 */
public class InvalidConfigurationException extends LibraryException {
    public InvalidConfigurationException(String message) {
        super(message);
    }
}

/**
 * Thrown when processing fails.
 */
public class ProcessingException extends LibraryException {
    public ProcessingException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

## Testing Pattern

```java
package com.company.library;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import static org.junit.jupiter.api.Assertions.*;

class LibraryTest {
    
    private Library library;
    
    @BeforeEach
    void setUp() {
        library = Library.builder()
            .timeout(Duration.ofSeconds(10))
            .retryCount(2)
            .build();
    }
    
    @Test
    void shouldProcessValidInput() {
        Input input = new Input("test data");
        Result result = library.process(input);
        assertNotNull(result);
    }
    
    @Test
    void shouldThrowExceptionForInvalidInput() {
        assertThrows(LibraryException.class, () -> {
            library.process(null);
        });
    }
}
```

## README Template

```markdown
# Library Name

Clear, one-line description of what this library does.

## Installation

### Maven

```xml
<dependency>
    <groupId>com.company</groupId>
    <artifactId>library-name</artifactId>
    <version>1.0.0</version>
</dependency>
```

### Gradle

```groovy
implementation 'com.company:library-name:1.0.0'
```

## Quick Start

```java
Library library = Library.builder()
    .timeout(Duration.ofSeconds(30))
    .retryCount(3)
    .build();

Result result = library.process(input);
```

## Configuration

### Standalone Usage

```java
Library library = Library.builder()
    .timeout(Duration.ofSeconds(30))
    .retryCount(3)
    .debugMode(true)
    .build();
```

### Spring Boot Integration

Add to `application.properties`:

```properties
library.timeout=30s
library.retry-count=3
library.debug-mode=false
```

The library will be auto-configured and available for injection:

```java
@Service
public class MyService {
    private final Library library;
    
    public MyService(Library library) {
        this.library = library;
    }
}
```

## Documentation

[Full documentation](https://github.com/company/library-name/wiki)

## License

MIT
```

## Key Principles

1. **Minimal dependencies**: Only depend on what you absolutely need
2. **Builder pattern**: Fluent API for configuration
3. **Immutability**: Make objects immutable after construction
4. **Clear exceptions**: Specific exception types for different error cases
5. **Logging**: Use SLF4J for logging, let users choose implementation
6. **Testing**: Comprehensive unit tests using JUnit 5
7. **Documentation**: Clear Javadoc and README with examples
8. **Spring Boot friendly**: Optional auto-configuration without requiring Spring
9. **Semantic versioning**: Follow semver for releases
10. **Thread safety**: Document thread-safety guarantees

## Anti-Patterns to Avoid

- ❌ Too many dependencies
- ❌ Tight coupling to frameworks
- ❌ Mutable state in library classes
- ❌ Complex inheritance hierarchies
- ❌ Reflection-heavy code
- ❌ Missing or poor documentation
- ❌ No tests
- ❌ Breaking changes in minor versions
- ❌ Global state or singletons (use DI instead)
- ❌ Checked exceptions for common cases

## Release Checklist

- [ ] All tests passing
- [ ] Checkstyle and SpotBugs clean
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version bumped appropriately
- [ ] Git tag created
- [ ] Deployed to Maven Central (or private repository)
