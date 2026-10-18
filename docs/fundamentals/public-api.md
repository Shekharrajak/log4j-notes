# The public logging API

Application code normally needs `LogManager`, `Logger`, levels, and parameterized messages. Add scoped context when multiple events belong to one request.

## One example to remember

```java
import org.apache.logging.log4j.CloseableThreadContext;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public final class OrderService {
    private static final Logger LOGGER = LogManager.getLogger(OrderService.class);

    public void createOrder(String requestId, String orderId) {
        try (CloseableThreadContext.Instance ignored = CloseableThreadContext
                .put("requestId", requestId)
                .put("orderId", orderId)) {
            LOGGER.info("Creating order {}", orderId);
        }
    }
}
```

`LogManager` obtains a logger; the class supplies its name. `INFO` supplies severity. The message pattern and argument describe what happened. Scoped context attaches request metadata and restores the preceding values when closed. A layout must include context fields to display them.

## Map questions to APIs

| Question | API |
| --- | --- |
| Where do I obtain a logger? | `LogManager` |
| Who is speaking? | `Logger` |
| How severe is this? | `Level` |
| What happened? | Parameterized string or `Message` |
| What category is this event? | `Marker` |
| Which request does it belong to? | `ThreadContext` |
| How do I restore metadata after a scope? | `CloseableThreadContext` |
| How do I assemble a fluent call? | `LogBuilder` |
| Why is logging itself failing? | `StatusLogger` diagnostics |

## Parameterization is not deferred Java evaluation

```java
LOGGER.debug("Order {}", buildExpensiveSummary());
```

Java calls `buildExpensiveSummary()` before entering the logger, even when DEBUG is disabled. Parameterization avoids eager string concatenation, but it cannot undo argument evaluation.

```java
if (LOGGER.isDebugEnabled()) {
    LOGGER.debug("Order {}", buildExpensiveSummary());
}
```

A supported supplier overload is another way to defer work. Check the overload and supplier type when mixing logging APIs.

## Preserve exceptions

```java
try {
    submitOrder();
} catch (RuntimeException failure) {
    LOGGER.error("Order submission failed", failure);
    throw failure;
}
```

Passing the exception preserves information that logging only `failure.getMessage()` would discard. This example rethrows it; logging is independent of the application's recovery policy.

The fluent equivalent of the logging call is:

```java
LOGGER.atError().withThrowable(failure).log("Order submission failed");
```

## API versus implementation

A reusable library can compile against `log4j-api`. The application supplies its logging implementation and configuration. Core-specific configuration and appender classes belong to the implementation side of that boundary.

## Recall

??? question "Will placeholders prevent an expensive argument method from running?"
    No. Use an enablement guard or an appropriate supplier overload.

??? question "Does CloseableThreadContext carry values into an executor task automatically?"
    No. Capture and establish the required context in the task, and restore it afterward.

## Source anchors

- [LogManager](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-api/src/main/java/org/apache/logging/log4j/LogManager.java): provider and context selection.
- [AbstractLogger](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-api/src/main/java/org/apache/logging/log4j/spi/AbstractLogger.java): overloads and enablement delegation.
- [CloseableThreadContext](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-api/src/main/java/org/apache/logging/log4j/CloseableThreadContext.java): scoped restoration.
- [LogBuilder](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-api/src/main/java/org/apache/logging/log4j/LogBuilder.java): fluent call contract.
