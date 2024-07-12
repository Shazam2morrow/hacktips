# Race Condition

## How to test for race condition vulnerabilities?

### Predict a potential collision

- **Is this endpoint security critical?**

    Many endpoints don't touch critical functionality, so they're not worth testing.

- **Is there any collision potential?**

    For a successful collision, you typically need two or more requests that trigger operations on the same record.

### Benchmark the behavior

ToDo

### Probe for clues



### Prove the concept

Sometimes rate limit is enforced per-username rather than per-session.

## Labs

- [Race Condition Vulnerabilities - PortSwigger](https://portswigger.net/web-security/race-conditions)

## Videos

- [DEF CON 31 - Smashing the State Machine the True Potential of Web Race Conditions - James Kettle](https://youtu.be/tKJzsaB1ZvI?feature=shared)
- [Race Conditions and How to Prevent Them - A Look at Dekker's Algorithm](https://youtu.be/MqnpIwN7dz0?feature=shared)

## References

- [Smashing the state machine: the true potential of web race conditions](https://portswigger.net/research/smashing-the-state-machine)