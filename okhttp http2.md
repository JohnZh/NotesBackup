# http2 连接健康判断

```java
public synchronized boolean isHealthy(long nowNs) {
  if (shutdown) return false;

  // A degraded pong is overdue.
  if (degradedPongsReceived < degradedPingsSent && nowNs >= degradedPongDeadlineNs) 
    return false;

  return true;
}
```

