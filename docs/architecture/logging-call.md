# One synchronous logging call

Trace `LOGGER.info("Created order {}", orderId)` through Log4j API and Core. This path assumes a normal synchronous Core logger; asynchronous loggers introduce a different handoff.

## The execution path

```text
AbstractLogger.info(...)
  -> logIfEnabled(...)
  -> Core Logger.isEnabled(...)
  -> message creation and logging delegation
  -> ReliabilityStrategy
  -> LoggerConfig creates and processes LogEvent
  -> AppenderControl
  -> Appender.append(event)
  -> Layout encodes event
  -> Manager writes output
```

## What happens at each boundary

| Boundary | Work |
| --- | --- |
| API -> Core logger | Check enablement; delegate an accepted call |
| Logger -> reliability strategy | Coordinate with configuration replacement |
| LoggerConfig -> event | Create event and attach event data |
| LoggerConfig -> appender controls | Apply event filters and route to appenders |
| Appender control -> appender | Check reference level/filter, recursion, lifecycle, and appender filters |
| Output appender -> manager | Encode and write; flush according to configuration and batch state |

Core's `Logger.PrivateConfig` caches the selected logger configuration and effective level. A configuration-wide filter can decide before the ordinary level comparison. Later filters can still reject an event.

`LoggerConfig` releases reusable event state after processing. An extension that retains events beyond a call must respect this lifecycle and make an appropriate stable copy.

`AbstractOutputStreamAppender` can encode directly to its manager or use a layout-produced byte array. A whole intermediate string is not mandatory for every output path.

## A routing example

Put this illustrative configuration in `log4j2.xml` on the application's classpath:

```xml
<Configuration>
  <Appenders>
    <Console name="Console">
      <PatternLayout pattern="%level %logger - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Logger name="com.shop" level="debug"/>
    <Root level="info">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

For logger `com.shop.OrderService`, `AbstractConfiguration.getLoggerConfig()` finds the `com.shop` rule by walking dot-separated parent names.

A DEBUG event passes that logger's threshold and reaches the root appender through additivity. The root's INFO logger level is **not reapplied** during parent event propagation. Appender-reference levels and filters can still reject it.

Attaching the same appender at both child and root can produce duplicate output. Setting `additivity="false"` stops parent propagation; a configuration with no appenders and additivity disabled has no output route.

## Recall

??? question "Will the example print a DEBUG call from com.shop.OrderService?"
    Yes, assuming no additional filters. The child enables DEBUG and propagation reaches the root's console appender.

??? question "What changes if com.shop has additivity=false and no appender references?"
    Its events have no appender route and do not reach the root appender.

??? question "Why does the call pass through a reliability strategy?"
    Logging must coordinate with configuration replacement and component shutdown.

## Source anchors and tests

- [Core Logger](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-core/src/main/java/org/apache/logging/log4j/core/Logger.java): `PrivateConfig` and reliability delegation.
- [LoggerConfig](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-core/src/main/java/org/apache/logging/log4j/core/config/LoggerConfig.java): `log`, `processLogEvent`, `logParent`.
- [AppenderControl](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-core/src/main/java/org/apache/logging/log4j/core/config/AppenderControl.java): checks around invocation.
- [AbstractOutputStreamAppender](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-core/src/main/java/org/apache/logging/log4j/core/appender/AbstractOutputStreamAppender.java): encoding paths.
- [NestedLoggerConfigTest](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-core-test/src/test/java/org/apache/logging/log4j/core/config/NestedLoggerConfigTest.java): inherited levels.
- [LoggerTest](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86/log4j-core-test/src/test/java/org/apache/logging/log4j/core/LoggerTest.java): additivity and other logger behavior.
