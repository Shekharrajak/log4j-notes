# Log4j Notes

Learn Log4j by following a logging call from application code to its destination. Start with the vocabulary, then connect each concept to the classes that implement it.

## Reading path

| Read | Question to answer |
| --- | --- |
| [Mental model](fundamentals/mental-model.md) | Which component owns each decision? |
| [Public API](fundamentals/public-api.md) | How does application code describe an event? |
| [Why SLF4J exists](fundamentals/why-slf4j.md) | Why do logging APIs and bridges coexist? |
| [One logging call](architecture/logging-call.md) | What happens inside Core after `logger.info()`? |

## Keep this distinction

```text
Application -> describes an event
Configuration -> selects processing and destinations
Implementation -> performs the work
```

A `Logger` is a reusable entry point. A `LogEvent` is one occurrence. A `LoggerConfig` controls routing. An `Appender` delivers events, and a `Layout` encodes them.

## How to use these notes

Read one page, hide it, and explain its flow from memory. Answer its recall questions before opening the answers. Follow a source link when the behavior seems surprising.

These are personal learning notes, not official Apache documentation. The initial source baseline is **Log4j 2.27.0-SNAPSHOT**, commit [`ae4117e25`](https://github.com/apache/logging-log4j2/blob/ae4117e25816346cfb749946e52e713fad397b86). Code links are pinned to that revision; release behavior may differ. The examples are illustrative, and the cited Java tests have been inspected rather than executed for these notes.

## Next topics

Configuration loading and reconfiguration, asynchronous logging, module reference pages, and output-prediction exercises will follow as we study them. Only completed notes appear in navigation.
