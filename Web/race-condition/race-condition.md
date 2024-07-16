# Race Condition

maybe things like transferring or withdrawing more funds than your account is supposed to have perhaps applying a single use discount code multiple times or bypassing a rate limit on a login form or such like one of my favorites that I found is I noticed that you can reuse a valid recapture solution

have something in common they could all be classed as limit overrun vulnerabilities they're all about doing

when changing email address sometimes Facebook would send two codes for two different addresses in a single confirmation email

Single-packet attack allows you make 20 to 30 requests arrive at the target server simultaneously completely regardless of network jitter.

Last-byte sync attack was invented before the single-packet attack and exploits the fact that the web server won't start to process a request until the whole request has arrived so by withholding the final bite and putting that in a separate packet you make the final packet of each request really small and kind of make things a little bit more reliable.

But with HTTP/2 you can stuff two entire requests into a single TCP packet and they were.

Nagel's algorithm.

The single-packet attack makes remote races local.

Web servers often delay requests that are sent too quickly and that means that you can send a single packet with a whole load of dummy requests in the middle of it which will cause a server side delay and mean that when your fast processing request at the end is reached it lines everything up perfectly so because with this everything is reaching the server at the same time and the server is injecting the delay for us network jitter is no longer going to make our attack fail.

## How to test for race condition vulnerabilities?

### Predict a potential collision

- **Is this endpoint security critical?**

    Many endpoints don't touch critical functionality, so they're not worth testing.

- **Is there any collision potential?**

    For a successful collision, you typically need two or more requests that trigger operations on the same record.

### Benchmark the behavior

Craft chaotic blend of conflicting requests.

Benchmark expected behavior:

- Send request blend in sequence.
- Analyze responses, timing, emails, side-effects.

### Probe for clues

- Send request blend in parallel.
- Look for anomalies.
- No anomalies? Tune timing to tighten execution spread.

### Prove the concept

Understand & Clean

- Trim superfluous requests.
- Tune the timing.
- Automate retries.

Explore impact

- Think of it as a structural weakness.
- Look for chains & variations.
- Don't stop at the first exploit.

Sometimes rate limit is enforced per-username rather than per-session.

# Examples

Object-masking via limit-overrun

A multi-endpoint collision

## Labs

- [Race Condition Vulnerabilities - PortSwigger](https://portswigger.net/web-security/race-conditions)

## Videos

- [Smashing the State Machine: The True Potential of Web Race Conditions](https://youtu.be/VzqG_-a8_Jo?feature=shared)
- [Race Conditions and How to Prevent Them - A Look at Dekker's Algorithm](https://youtu.be/MqnpIwN7dz0?feature=shared)

## References

- [Smashing the state machine: the true potential of web race conditions](https://portswigger.net/research/smashing-the-state-machine)
