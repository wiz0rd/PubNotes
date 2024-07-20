# Local Development Logging Strategy for POS System

## Goal
Implement a simple, effective logging solution for the POS system that works well in a local development environment without incurring any additional costs.

## Technologies
1. Go's standard library `log` package or a lightweight logging library like `logrus`
2. Log rotation using `lumberjack`
3. Basic log analysis using command-line tools

## Implementation Steps

### 1. Set up Logging in Go

Option A: Using standard library (simplest approach)

```go
package main

import (
    "log"
    "os"
)

func main() {
    // Open a file for logging
    f, err := os.OpenFile("pos_system.log", os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalf("error opening file: %v", err)
    }
    defer f.Close()

    // Set log output to file
    log.SetOutput(f)

    // Example log entries
    log.Println("POS system started")
    log.Printf("Sale completed: Amount: $%.2f, Items: %d", 25.50, 3)
}
```

Option B: Using logrus (more features, still lightweight)

```go
package main

import (
    "os"
    "github.com/sirupsen/logrus"
)

var log = logrus.New()

func main() {
    // Open a file for logging
    f, err := os.OpenFile("pos_system.log", os.O_RDWR|os.O_CREATE|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalf("error opening file: %v", err)
    }
    defer f.Close()

    // Configure logrus
    log.SetOutput(f)
    log.SetFormatter(&logrus.JSONFormatter{})
    log.SetLevel(logrus.InfoLevel)

    // Example log entries
    log.Info("POS system started")
    log.WithFields(logrus.Fields{
        "amount": 25.50,
        "items":  3,
    }).Info("Sale completed")
}
```

### 2. Implement Log Rotation

To prevent log files from growing too large, use the `lumberjack` package for log rotation:

```go
package main

import (
    "github.com/sirupsen/logrus"
    "gopkg.in/natefinch/lumberjack.v2"
)

var log = logrus.New()

func main() {
    // Configure log rotation
    log.SetOutput(&lumberjack.Logger{
        Filename:   "pos_system.log",
        MaxSize:    100, // megabytes
        MaxBackups: 3,
        MaxAge:     28, // days
        Compress:   true,
    })

    log.SetFormatter(&logrus.JSONFormatter{})
    log.SetLevel(logrus.InfoLevel)

    // Your POS system logic here
}
```

### 3. Basic Log Analysis

For local development, you can use command-line tools to analyze logs:

1. View logs in real-time:
   ```
   tail -f pos_system.log
   ```

2. Search for specific events:
   ```
   grep "Sale completed" pos_system.log
   ```

3. Count occurrences of an event:
   ```
   grep "Sale completed" pos_system.log | wc -l
   ```

4. Analyze JSON-formatted logs (if using logrus with JSONFormatter):
   ```
   cat pos_system.log | jq '.'
   ```

### 4. Error Handling and Reporting

Implement a simple error logging function:

```go
func logError(err error, context string) {
    log.WithFields(logrus.Fields{
        "error":   err.Error(),
        "context": context,
    }).Error("An error occurred")
}

// Usage
if err != nil {
    logError(err, "Failed to process sale")
}
```

### 5. Performance Logging

Add basic performance logging:

```go
func logPerformance(operation string, duration time.Duration) {
    log.WithFields(logrus.Fields{
        "operation": operation,
        "duration":  duration.Milliseconds(),
    }).Info("Performance metric")
}

// Usage
start := time.Now()
// ... perform operation ...
logPerformance("process_sale", time.Since(start))
```

## Best Practices for Local Logging

1. Log meaningful events: sales, errors, system start/stop, etc.
2. Use log levels appropriately (INFO for normal operations, ERROR for issues, DEBUG for development details)
3. Include relevant context in log messages (user IDs, transaction IDs, etc.)
4. Regularly review logs during development to catch issues early
5. Implement a log retention policy to manage disk space
6. Consider security: don't log sensitive information like full credit card numbers or passwords

