---
name: spring-boot-style
description: Write Java and Spring Boot code following modern Spring Boot best practices. Use this skill when writing Java code, Spring Boot applications, creating services, controllers, repositories, or any Java file. Triggers on Java/Spring Boot code generation, refactoring requests, code review, or when the user mentions Spring Boot, Spring Framework, or enterprise Java patterns. Embodies REST best practices, clean architecture, dependency injection, and the "convention over configuration" philosophy.
---

# Spring Boot Style Guide

Write Java and Spring Boot code following best practices: **convention over configuration**, **clean architecture**, **testability** above all.

## Quick Reference

### REST Controller Structure
- **Use @RestController**: Combines @Controller and @ResponseBody
- **HTTP methods map to CRUD**: GET (read), POST (create), PUT (update), DELETE (delete)
- **Keep controllers thin**: Delegate to services
- **Return ResponseEntity for control**: Or use @ResponseStatus for simple cases

```java
package com.company.api.controller;

import com.company.api.dto.MessageDto;
import com.company.api.service.MessageService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/messages")
@RequiredArgsConstructor
public class MessageController {
    
    private final MessageService messageService;
    
    @GetMapping
    public ResponseEntity<List<MessageDto>> getAllMessages() {
        return ResponseEntity.ok(messageService.findAll());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<MessageDto> getMessage(@PathVariable Long id) {
        return messageService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<MessageDto> createMessage(@Valid @RequestBody MessageDto dto) {
        MessageDto created = messageService.create(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<MessageDto> updateMessage(
            @PathVariable Long id,
            @Valid @RequestBody MessageDto dto) {
        return messageService.update(id, dto)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteMessage(@PathVariable Long id) {
        messageService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Service Layer Pattern
- **Business logic lives here**: Not in controllers or repositories
- **Use @Transactional**: At service methods that modify data
- **Constructor injection**: With final fields and @RequiredArgsConstructor
- **Return Optional**: For single results that might not exist

```java
package com.company.api.service;

import com.company.api.dto.MessageDto;
import com.company.api.entity.Message;
import com.company.api.repository.MessageRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class MessageService {
    
    private final MessageRepository messageRepository;
    private final MessageMapper messageMapper;
    
    @Transactional(readOnly = true)
    public List<MessageDto> findAll() {
        return messageRepository.findAll()
            .stream()
            .map(messageMapper::toDto)
            .collect(Collectors.toList());
    }
    
    @Transactional(readOnly = true)
    public Optional<MessageDto> findById(Long id) {
        return messageRepository.findById(id)
            .map(messageMapper::toDto);
    }
    
    @Transactional
    public MessageDto create(MessageDto dto) {
        Message message = messageMapper.toEntity(dto);
        Message saved = messageRepository.save(message);
        return messageMapper.toDto(saved);
    }
    
    @Transactional
    public Optional<MessageDto> update(Long id, MessageDto dto) {
        return messageRepository.findById(id)
            .map(message -> {
                messageMapper.updateEntity(dto, message);
                return messageMapper.toDto(message);
            });
    }
    
    @Transactional
    public void delete(Long id) {
        messageRepository.deleteById(id);
    }
}
```

### Repository Layer
- **Extend JpaRepository**: Gets CRUD methods for free
- **Custom queries with @Query**: When method names get too long
- **Avoid N+1 queries**: Use @EntityGraph or JOIN FETCH

```java
package com.company.api.repository;

import com.company.api.entity.Message;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface MessageRepository extends JpaRepository<Message, Long> {
    
    // Spring Data JPA method name query
    List<Message> findByAuthorId(Long authorId);
    
    // Custom query with JOIN FETCH to avoid N+1
    @Query("SELECT m FROM Message m JOIN FETCH m.author WHERE m.id = :id")
    Optional<Message> findByIdWithAuthor(Long id);
    
    // Native query when needed
    @Query(value = "SELECT * FROM messages WHERE created_at > NOW() - INTERVAL 1 DAY", 
           nativeQuery = true)
    List<Message> findRecentMessages();
}
```

### Entity Classes
- **JPA annotations on fields**: Or getters, be consistent
- **Use Lombok**: Reduce boilerplate with @Data, @Entity, @Table
- **Relationships clearly defined**: @ManyToOne, @OneToMany with proper fetch types
- **Audit fields**: Use @CreatedDate and @LastModifiedDate with @EntityListeners

```java
package com.company.api.entity;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@Entity
@Table(name = "messages")
@Data
@NoArgsConstructor
@EntityListeners(AuditingEntityListener.class)
public class Message {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 1000)
    private String content;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;
    
    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;
}
```

### DTO Pattern
- **Separate DTOs from entities**: Don't expose entities in API
- **Validation annotations**: Use @Valid in controllers
- **Immutable when possible**: Use records (Java 16+) or @Value from Lombok

```java
package com.company.api.dto;

// Note: Spring Boot 3.x uses Jakarta EE (jakarta.validation.*) instead of javax.validation.*
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;

@Data
public class MessageDto {
    
    private Long id;
    
    @NotBlank(message = "Content is required")
    @Size(max = 1000, message = "Content must be less than 1000 characters")
    private String content;
    
    private Long authorId;
    private String authorName;
    
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

// Or using Java records (Java 16+)
public record MessageDto(
    Long id,
    @NotBlank String content,
    Long authorId,
    String authorName,
    LocalDateTime createdAt,
    LocalDateTime updatedAt
) {}
```

### Exception Handling
- **@ControllerAdvice**: Global exception handler
- **Specific exceptions**: Domain-specific exceptions with clear messages
- **Standard error response**: Consistent error format

```java
package com.company.api.exception;

import lombok.AllArgsConstructor;
import lombok.Data;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.joining(", "));
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            message,
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
    
    @Data
    @AllArgsConstructor
    static class ErrorResponse {
        private int status;
        private String message;
        private LocalDateTime timestamp;
    }
}

// Custom exception
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### Configuration Classes
- **@Configuration**: For beans that need complex initialization
- **@ConfigurationProperties**: Type-safe configuration from application.properties
- **Profiles**: Use @Profile for environment-specific beans

```java
package com.company.api.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
public class AppConfiguration {
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource(DatabaseProperties props) {
        // Production datasource configuration
        return DataSourceBuilder.create()
            .url(props.getUrl())
            .username(props.getUsername())
            .password(props.getPassword())
            .build();
    }
}

@ConfigurationProperties(prefix = "app.database")
@Data
public class DatabaseProperties {
    private String url;
    private String username;
    private String password;
    private int maxPoolSize = 10;
}
```

### Testing Pattern
- **@SpringBootTest**: For integration tests
- **@WebMvcTest**: For controller tests
- **@DataJpaTest**: For repository tests
- **MockMvc**: For testing REST endpoints
- **Testcontainers**: For database integration tests

```java
package com.company.api.controller;

import com.company.api.dto.MessageDto;
import com.company.api.service.MessageService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.util.Optional;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(MessageController.class)
class MessageControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    @MockBean
    private MessageService messageService;
    
    @Test
    void shouldCreateMessage() throws Exception {
        MessageDto dto = new MessageDto();
        dto.setContent("Test message");
        
        when(messageService.create(any())).thenReturn(dto);
        
        mockMvc.perform(post("/api/messages")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.content").value("Test message"));
    }
    
    @Test
    void shouldReturnNotFoundForNonexistentMessage() throws Exception {
        when(messageService.findById(999L)).thenReturn(Optional.empty());
        
        mockMvc.perform(get("/api/messages/999"))
                .andExpect(status().isNotFound());
    }
}
```

## Core Principles

1. **Convention over configuration**: Use Spring Boot defaults
2. **Dependency injection**: Constructor injection with final fields
3. **Layer separation**: Controllers → Services → Repositories → Entities
4. **DTO pattern**: Don't expose entities in APIs
5. **Immutability**: Use final, records, and immutable collections
6. **Fail fast**: Validate early with @Valid
7. **Clean exceptions**: Use @ControllerAdvice and specific exceptions
8. **Testing**: Write tests at each layer
9. **Logging**: Use SLF4J through Lombok's @Slf4j
10. **Security**: Use Spring Security, never roll your own

## Anti-Patterns to Avoid

- ❌ Field injection instead of constructor injection
- ❌ Logic in controllers (should be in services)
- ❌ Exposing entities directly in REST APIs
- ❌ Manual transaction management
- ❌ Ignoring N+1 query problems
- ❌ Not using @Transactional(readOnly = true) for queries
- ❌ Catching generic Exception
- ❌ Not using Spring Boot starters
- ❌ Manual bean wiring instead of component scanning
- ❌ Not writing tests

## Package Structure

```
com.company.api/
├── ApiApplication.java          # Main class
├── config/                      # Configuration classes
├── controller/                  # REST controllers
├── service/                     # Business logic
├── repository/                  # Data access
├── entity/                      # JPA entities
├── dto/                         # Data transfer objects
├── mapper/                      # Entity ↔ DTO mappers
├── exception/                   # Custom exceptions
└── util/                        # Utility classes
```

## application.properties Example

```properties
# Server
server.port=8080
server.servlet.context-path=/api

# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=user
spring.datasource.password=password

# JPA
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.com.company=INFO
logging.level.org.springframework.web=DEBUG

# Actuator
management.endpoints.web.exposure.include=health,info,metrics
```

Remember: Spring Boot is opinionated. Follow the conventions and your code will be cleaner, easier to test, and more maintainable.
