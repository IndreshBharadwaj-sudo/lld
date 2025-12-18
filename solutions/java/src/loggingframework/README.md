# Logging Framework (LLD)

## Problem Statement

Design and implement a flexible and extensible logging framework that can be used by applications to log messages at different levels (INFO, DEBUG, ERROR, etc.), support multiple output destinations (console, file, etc.), and allow for custom formatting of log messages.

---

## Requirements

- **Log Levels:** Support for multiple log levels (INFO, DEBUG, ERROR, etc.).
- **Multiple Appenders:** Ability to log to different destinations (console, file, etc.).
- **Custom Formatting:** Support for custom log message formatting.
- **Configuration:** Ability to configure loggers and appenders.
- **Thread Safety:** Should be thread-safe for concurrent logging.
- **Extensibility:** Easy to add new log levels, appenders, or formatters.

---

## Core Entities

- **Logger:** Main class used by clients to log messages.
- **LogLevel:** Enum representing different log levels.
- **LogMessage:** Encapsulates the details of a log event.
- **LogFormatter:** Interface for formatting log messages.
- **DefaultFormatter:** Default implementation of `LogFormatter`.
- **LoggerConfig:** Holds configuration for the logger (appenders, formatters, etc.).
- **LogAppender (in `logappender/`):** Interface and implementations for output destinations (e.g., ConsoleAppender, FileAppender).

---

## Class Design

## UML Class Diagram

![](../../../../uml-diagrams/class-diagrams/loggingframework-class-diagram.png)

### 1. Logger
- **Methods:**
  - `log(LogLevel level, String message)`
  - `info(String message)`
  - `debug(String message)`
  - `error(String message)`
  - `setConfig(LoggerConfig config)`

### 2. LogLevel
- Enum for log levels (INFO, DEBUG, ERROR, etc.)

### 3. LogMessage
- Fields: `level`, `message`, `timestamp`, etc.

### 4. LogFormatter (Interface)
- `String format(LogMessage message)`

### 5. DefaultFormatter
- Implements `LogFormatter` with a default format.

### 6. LoggerConfig
- Holds configuration for loggers (appenders, formatters, log level).

### 7. LogAppender (in `logappender/`)
- Interface for appenders.
- Implementations: `ConsoleAppender`, `FileAppender`, etc.

---

## Design Patterns Used

- **Strategy Pattern:** For interchangeable log formatters and appenders.
- **Singleton Pattern:** (If used) For global logger instance.
- **Factory Pattern:** (Optional) For creating appenders/formatters based on config.
- **Observer Pattern:** (Conceptually, for notifying multiple appenders.)

---

## Example Usage

```java
Logger logger = new Logger();
logger.setConfig(new LoggerConfig(...));
logger.info("Application started");
logger.error("An error occurred");
```

---

## Demo

See `LoggingFrameworkDemo.java` for a sample usage of the logging framework.

---

## Extending the Framework

- **Add a new log level:** Update `LogLevel.java`.
- **Add a new appender:** Implement the `LogAppender` interface in `logappender/`.
- **Add a new formatter:** Implement the `LogFormatter` interface.

---

≥≥≥≥ How does exceutor service helps in these can you explain how will it behave in mutiple threads
Why single thread?

Log ordering must be preserved

No race conditions in appenders

Deterministic output

Excellent question.
This is exactly the depth interviewers expect when you say “we use ExecutorService for async logging”.

I’ll explain this from first principles, then tie each point directly to your bullets.

1️⃣ What actually happens when MULTIPLE threads log

Assume you have many application threads:

Thread A → logger.info("A1")
Thread B → logger.info("B1")
Thread C → logger.info("C1")


All of them call:

asyncLogProcessor.process(logEvent)

Without ExecutorService (bad approach)

If each call did:

new Thread(() -> writeLog()).start();

Problems

Unbounded threads → resource exhaustion

Writes happen in random order

Appenders may interleave writes

Race conditions everywhere

Example output:

A1
C1
B1
A2


(Non-deterministic, impossible to debug)

2️⃣ What ExecutorService ACTUALLY does

An ExecutorService has two important parts:

Task Queue

Worker Threads

When a thread submits a task
executor.submit(() -> processLog(event));


The task is enqueued

The calling thread returns immediately

Worker thread later picks up the task

This is the Producer–Consumer pattern.

3️⃣ What changes with a SINGLE-THREAD Executor
ExecutorService executor =
    Executors.newSingleThreadExecutor();


This creates:

✅ ONE background worker thread

✅ FIFO task queue

Let’s replay the same logging scenario
Application threads (producers)
Thread A → submit(A1)
Thread B → submit(B1)
Thread C → submit(C1)

Inside ExecutorService
Queue: A1 → B1 → C1
Worker Thread:
  take A1 → process
  take B1 → process
  take C1 → process

4️⃣ Now your three bullet points explained EXACTLY
✅ 1️⃣ Why log ordering must be preserved

Logs are time-ordered facts.

If this happens:

UserLoggedIn
UserLoggedOut


You MUST NOT see:

UserLoggedOut
UserLoggedIn

Single-thread executor guarantees:

Tasks execute in submission order

FIFO queue preserves order

One worker → no reordering

➡️ This gives strict log ordering

This is crucial for debugging & auditing.

✅ 2️⃣ Why no race conditions in appenders

Your appenders do things like:

fileWriter.write(...)
console.print(...)


If multiple threads do this concurrently:

File writes interleave

Logs get corrupted

Partial lines appear

Single-thread executor guarantees:

Only ONE thread calls appender.append()

No need for synchronized

No shared-state corruption

➡️ Appender code stays simple and fast

Real logging frameworks do the same.

✅ 3️⃣ Why deterministic output

Deterministic means:

Same inputs → same outputs

With single-thread executor:

Same submission order

Same processing order

Same final logs

Without it:

Thread scheduling randomness

OS-dependent behavior

Impossible-to-reproduce bugs

Deterministic logs = debuggable system.

5️⃣ Why NOT multiple threads in executor?
Suppose:
Executors.newFixedThreadPool(4)

What breaks

Task A and B run in parallel

B may finish before A

Ordering breaks

Appenders need locks

Performance drops due to contention

You’d need:

synchronized(fileWriter) { ... }


Which defeats async benefits.