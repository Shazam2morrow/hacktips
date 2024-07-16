# Race Conditions

maybe things like transferring or withdrawing more funds than your account is supposed to have perhaps applying a single use discount code multiple times or bypassing a rate limit on a login form or such like one of my favorites that I found is I noticed that you can reuse a valid recapture solution

have something in common they could all be classed as limit overrun vulnerabilities they're all about doing

when changing email address sometimes Facebook would send two codes for two different addresses in a single confirmation email

Single-packet attack allows you make 20 to 30 requests arrive at the target server simultaneously completely regardless of network jitter.

Last-byte sync attack was invented before the single-packet attack and exploits the fact that the web server won't start to process a request until the whole request has arrived so by withholding the final bite and putting that in a separate packet you make the final packet of each request really small and kind of make things a little bit more reliable.

But with HTTP/2 you can stuff two entire requests into a single TCP packet and they were.

Nagel's algorithm.

The single-packet attack makes remote races local.

Web servers often delay requests that are sent too quickly and that means that you can send a single packet with a whole load of dummy requests in the middle of it which will cause a server side delay and mean that when your fast processing request at the end is reached it lines everything up perfectly so because with this everything is reaching the server at the same time and the server is injecting the delay for us network jitter is no longer going to make our attack fail.

## What are race condition vulnerabilities?

Race conditions are a common type of vulnerability related to business logic flaws. They happen when a system or application handles multiple requests at the same time without proper safeguards. This allows different threads to interact with the same data simultaneously, causing unintended behavior.

A race condition attack exploits this by sending carefully timed requests to create collisions and take advantage of the resulting unpredictable behavior for malicious purposes.

## How do race condition vulnerabilities arise?

Race condition vulnerabilities arise when a system or application processes multiple threads or requests concurrently without proper synchronization. These vulnerabilities can be exploited by attackers who craft and time their requests to trigger the race condition, leading to unintended and often harmful behavior in the system:

1. **Shared Resource Access**: When multiple threads or processes access shared resources (e.g., files, variables, or memory) simultaneously without proper locking mechanisms, it can cause inconsistent or corrupted data states.

2. **Time of Check to Time of Use (TOCTOU)**: This occurs when there is a gap between checking a condition (e.g., verifying user permissions) and using the result of that check (e.g., performing an action based on those permissions). An attacker can exploit this gap to change the conditions and bypass security checks.

3. **Inadequate Locking Mechanisms**: When the system does not implement adequate locking or synchronization mechanisms, it can lead to multiple threads modifying the same data concurrently, causing data corruption or unintended behavior.

4. **Poorly Designed Code**: Race conditions often arise from poorly designed code where the developer did not anticipate concurrent execution or failed to implement proper synchronization mechanisms.

## What is the impact of race condition vulnerabilities?

The impact of race condition vulnerabilities can be significant and varied, depending on the context in which they occur. Here are some potential impacts:

1. **Data Corruption**: concurrent access to shared resources without proper synchronization can lead to data inconsistencies or corruption, affecting the integrity of the system.

2. **Security Bypasses**: attackers can exploit race conditions to bypass security mechanisms, such as authentication and authorization checks, leading to unauthorized access to sensitive data or system functionalities. For example, bypassing an anti-bruteforce rate-limit or reusing a single CAPTCHA solution multiples times.

3. **Business Logic Bypasses**: exploiting race conditions can allow attackers to bypass some business logic like redeeming a gift card multiple times, repeatedly applying a single discount code or rating a product multiple times.

4. **Denial of Service (DoS)**: race conditions can cause systems to behave unpredictably or crash, leading to denial of service, where legitimate users are unable to access the system or service.

5. **Financial Loss**: in applications handling financial transactions, race conditions can result in incorrect transactions, duplicate charges, or unauthorized fund transfers, leading to financial losses for users and organizations. For example, withdrawing or transferring cash in excess of your account balance.

6. **Privilege Escalation**: exploiting race conditions can allow attackers to escalate their privileges, gaining higher-level access and control over the system than intended.

7. **Code Execution**: in some cases, race conditions can be exploited to execute arbitrary code, potentially allowing attackers to take full control of the affected system.

## How to test for race condition vulnerabilities?

The primary challenge when testing for race conditions is timing the requests so that at least two race windows line up, causing a collision. This window is often just milliseconds and can be even shorter.

Even if you send all of the requests at exactly the same time, in practice there are various uncontrollable and unpredictable external factors that affect when the server processes each request and in which order.

### Factors that affect the exploitation of race condition vulnerabilities

The image below illustrates how two parallel requests sent at the same time experience different stages of latency:

![https://portswigger.net/web-security/race-conditions/images/race-conditions-basic.png](race-conditions-basic.png)

The critical area, marked as the "race window," shows where the timing overlap occurs, leading to a race condition. The red exclamation marks highlight the points where the race condition can cause unintended behavior due to concurrent data access.

#### Network latency

Network latency refers to the time it takes for data to travel from its source to its destination across a network. It's usually measured in milliseconds (ms). Imagine you're sending a message to a friend in another country via a messenger. Network latency is the time from when you press "send" on your phone until your friend's phone receives the message. If the network latency is high, there might be a noticeable delay before your friend sees the message.

#### Jitter

Jitter involves the variation in packet arrival time. It's a measure of the consistency of latency. In simpler terms, if latency is how long it takes for packets to reach their destination, jitter is how much those times vary from packet to packet. Consider a video call. If every piece of data (video frame, audio packet) arrives at a regular interval, you'll see a smooth conversation. But if some packets get delayed more than others (because of varying latencies), you'll notice glitches or interruptions in the video or audio. This inconsistency is jitter.

#### Internal latency

Internal latency refers specifically to delays within a network system or device, rather than delays caused by the physical distance the data must travel. This could be due to processing times within network devices like routers or switches. When you access a website, your data passes through several devices on your local network before reaching the internet. If your router is old or overwhelmed with traffic, it might slow down the processing of your data, adding to the overall latency. This delay caused by your router is a type of internal latency.

#### Race window

The period of time during which a collision is possible is known as the "race window". This could be the fraction of a second between two interactions with the database, for example.

### How to overcome the factors that affect the exploitation of race condition vulnerabilities?

Basically, there are two ways to overcome network latencies: **last-byte synchronization** and **single-packet attack**.

#### Last-byte synchronization

Last-byte synchronization is a technique used to test for race condition vulnerabilities by ensuring that multiple threads or requests reach the critical point of their execution simultaneously. This is done by **synchronizing the sending of the last byte of each request so that they are processed at the same time**.

The main drawback is that this method requires precise timing, which can be difficult to achieve consistently due to variations in network latency and system processing times. This makes it less reliable for detecting race conditions under varying real-world conditions like jitter.

#### Single-packet attack

The single-packet attack enables you to completely neutralize interference from network jitter by using a single TCP packet to complete 20-30 requests simultaneously. Although you can often use just two requests to trigger an exploit, sending a large number of requests like this helps to mitigate internal latency, also known as server-side jitter.

This attack is 4 to 10 times more effective than the last-byte synchronization. This is possible due to the fact that HTTP/2 allows HTTP requests to be sent over a single connection concurrently, whereas in HTTP/1.1 they have to be sequential.

### Detecting and exploiting limit overrun race conditions with Burp Suite

#### Burp Repeater

Burp Suite (starting from version 2023.9) adds powerful new capabilities to Burp Repeater that enable you to easily send a group of parallel requests in a way that greatly reduces the impact of the network jitter.

To send a group of requests:

1. Create a Repeater tab group and add the relevant tabs to it. Alternatively, you can create a group and duplicate tabs within it. This can be helpful if you're testing for race condition vulnerabilities as it makes the process of creating identical requests much more efficient.

2. Select one of the tabs in the group.

3. Click the drop-down arrow by the side of the Send button and select one of the available options:
    - Send group in sequence (single connection).
    - Send group in sequence (separate connections).
    - Send group in parallel.

4. Click Send group. Repeater sends all of the requests from the grouped tabs.

##### Sending requests in sequence

You can send requests in sequence using either a single connection or multiple connections. To send a sequence of requests, the group must meet the following criteria:

- There must not be any WebSocket message tabs in the group.
- There must not be any empty tabs in the group.

###### Sending over a single connection

Repeater establishes a connection to the target, sends the requests from all of the tabs in the group, and then closes the connection.

Sending requests over a single connection enables you to test for potential client-side desync vectors. It also reduces the jitter that can occur when establishing TCP connections. This is useful for timing-based attacks that rely on being able to compare responses with very small differences in timings.

There are also some additional criteria to send over a single connection:

- All tabs must have the same target.
- All tabs must use the same HTTP version (that is, they must either all use HTTP/1 or all use HTTP/2).

###### Sending over separate connections

Repeater establishes a connection to the target, sends the request from the first tab, and then closes the connection. It repeats this process for all of the other tabs in the order they are arranged in the group.

Sending requests over separate connections makes it easier to test for vulnerabilities that require a multi-step process.

##### Sending requests in parallel

Repeater sends the requests from all of the group's tabs at once. This is useful as a way to identify and exploit race conditions.

Repeater synchronizes parallel requests to ensure that they all arrive in full at the same time. It uses different synchronization techniques depending on the HTTP version used:

- When sending over HTTP/1, it uses last-byte synchronization. This is where multiple requests are sent over concurrent connections, but the last byte of each request in the group is withheld. After a short delay, these last bytes are sent down each connection simultaneously.
- When sending over HTTP/2+, Repeater sends the group using a single packet attack. This is where multiple requests are sent via a single TCP packet.

When you select a tab containing a response to a parallel request, an indicator in the bottom-right corner displays the order in which that response was received within the group (for example, 1/3, 2/3).

Also you cannot send macro requests in parallel. This is to prevent macros from interfering with request synchronization.

To send a group of requests in parallel, the group must meet the following criteria:

1. All requests in the group must use the same host, port, and transport layer protocols.
2. HTTP/1 keep-alive must not be enabled for the project.

#### Turbo Intruder

Turbo Intruder is Burp Suite's extension that requires some proficiency in Python, but is suited to more complex attacks, such as ones that require multiple retries, staggered request timing, or an extremely large number of requests.

To use the single-packet attack in Turbo Intruder:

1. Ensure that the target supports `HTTP/2`. The single-packet attack is incompatible with `HTTP/1`.

2. Set the `engine=Engine.BURP2` and `concurrentConnections=1` configuration options for the request engine.

3. When queueing your requests, group them by assigning them to a named gate using the gate argument for the `engine.queue()` method.

4. To send all of the requests in a given group, open the respective gate with the `engine.openGate()` method.

For example it can look like this:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint concurrentConnections=1, engine=Engine.BURP2)
    
    # queue 20 requests in gate '1'
    for i in range(20):
        engine.queue(target.req, gate='1')
    
    # send all requests in gate '1' in parallel
    engine.openGate('1')
```

For more details, see the `race-single-packet-attack.py` template provided in Turbo Intruder's default examples directory.

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

## How to prevent race condition vulnerabilities?

ToDo

## Interactive checklist

ToDo

## Labs

- [Race Condition Vulnerabilities - PortSwigger](https://portswigger.net/web-security/all-labs#race-conditions)

## Videos

- [Smashing the State Machine: The True Potential of Web Race Conditions](https://youtu.be/VzqG_-a8_Jo?feature=shared)
- [Race Conditions and How to Prevent Them - A Look at Dekker's Algorithm](https://youtu.be/MqnpIwN7dz0?feature=shared)

## References

- [Smashing the state machine: the true potential of web race conditions](https://portswigger.net/research/smashing-the-state-machine)
- [Cracking reCAPTCHA, Turbo Intruder style](https://portswigger.net/research/cracking-recaptcha-turbo-intruder-style)
- [Race conditions on the web](https://www.josipfranjkovic.com/blog/race-conditions-on-web)
- [Timeless Timing Attacks: Exploiting Concurrency to Leak Secrets over Remote Connections](https://www.usenix.org/conference/usenixsecurity20/presentation/van-goethem)
- [The single-packet attack: making remote race-conditions 'local'](https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local)
- [Race Condition Vulnerabilities - PortSwigger](https://portswigger.net/web-security/race-conditions)
- [Sending grouped HTTP requests](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group)
