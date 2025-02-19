# Rate Limiting Strategies

**Rate Limiting** is a technique used to control the amount of incoming requests to a service in a given period, protecting it from overuse, abuse, or malicious attacks. In the context of an API Gateway, rate limiting helps ensure fair usage, prevent resource exhaustion, and improve the overall stability of a microservices ecosystem.

There are several **rate limiting strategies** that can be used to control traffic, each with its own design and use case. Below are the most commonly used strategies, along with their detailed explanations.

---

### **1. Fixed Window Rate Limiting**

**Design**: 
- This strategy is based on dividing time into fixed "windows" (e.g., 1 minute, 1 hour).
- Each window has a predefined limit (e.g., 1000 requests per minute). When a client exceeds this limit within the window, it is blocked until the window resets.
- Once the time window ends (e.g., after one minute), the request count is reset, and the client can make requests again.

**Example**:
- Limit: 1000 requests per minute.
- If a user makes 1001 requests in the first minute, the 1001st request will be blocked, and the limit resets at the start of the next minute.

**Pros**:
- Simple to implement.
- Easy to understand and manage.

**Cons**:
- Can cause burst traffic at the start of a new window (i.e., clients may make 1000 requests right at the beginning of the window, leading to spikes in traffic).
- Can lead to unfair traffic distribution (i.e., a client that makes 999 requests at the start of the window can make 999 more right after the reset).

---

### **2. Sliding Window Rate Limiting**

**Design**:
- This strategy is an improvement over fixed window rate limiting.
- Instead of resetting the count at the start of each fixed window, the sliding window approach keeps track of the actual timestamps of requests.
- The rate limit is applied to a "window" of time (e.g., the last 60 seconds), where the system dynamically calculates the number of requests in that time frame and enforces the rate limit.
- This allows for more gradual request distribution.

**Example**:
- Limit: 1000 requests per 60 seconds.
- The system tracks the actual timestamps of each request and checks how many requests were made in the last 60 seconds before accepting the current one.

**Pros**:
- Prevents traffic spikes at the start of a new window.
- More granular control over rate limiting.

**Cons**:
- Slightly more complex to implement than fixed window rate limiting.
- Requires tracking request timestamps for each client.

---

### **3. Token Bucket Rate Limiting**

**Design**:
- The token bucket algorithm allows clients to make requests as long as there are tokens in the "bucket."
- Tokens are added to the bucket at a fixed rate (e.g., one token per second). Each request consumes one token, and if the bucket is empty, requests are rejected.
- The key feature of this approach is that it allows for **bursty traffic**. If the client hasn’t made requests for a while, the bucket accumulates tokens, allowing it to make multiple requests at once, up to the capacity of the bucket.
- However, requests cannot be made once the bucket is empty, even if the rate limit is temporarily exceeded.

**Example**:
- Limit: 10 requests per minute (1 token per 6 seconds).
- The bucket holds up to 10 tokens. If the client hasn’t made a request in a while, tokens accumulate, and it can burst up to 10 requests quickly, but after the tokens run out, it needs to wait for new tokens to accumulate.

**Pros**:
- Allows burst traffic without violating the rate limit.
- Provides a good balance between controlling traffic and allowing occasional spikes.

**Cons**:
- Slightly more complex to implement than fixed/sliding window.
- May still be unfair in cases where a client consumes all tokens quickly and then has to wait for replenishment.

---

### **4. Leaky Bucket Rate Limiting**

**Design**:
- The leaky bucket algorithm is similar to the token bucket, but it has a fixed rate at which requests can "leak" out of the bucket.
- The system processes requests at a constant rate (e.g., 1 request per second), regardless of bursts.
- Excessive requests are discarded, or queued in the bucket. Once the bucket is full, further requests are dropped until there’s space in the bucket for new requests.

**Example**:
- Limit: 1 request per second.
- Requests are processed at a constant rate, so if more than one request arrives per second, the excess will be discarded or delayed until the bucket has space.

**Pros**:
- Ensures smooth, steady processing of requests.
- Works well for controlling consistent traffic flow.

**Cons**:
- Less flexible for burst traffic.
- Requests can be discarded when the bucket overflows, leading to possible denial of service during high traffic periods.

---

### **5. Concurrent Request Limiting**

**Design**:
- This strategy limits the **number of simultaneous (concurrent) requests** a client can make to a service.
- For example, a client may be allowed to make only 5 simultaneous API requests at a time. Any additional requests are queued or rejected until one of the active requests completes.

**Example**:
- Limit: 5 concurrent requests.
- If the client makes 6 requests simultaneously, the 6th request will be rejected until one of the first 5 finishes.

**Pros**:
- Helps avoid overloading the service with too many simultaneous requests.
- Good for applications that need to handle a limited amount of resources or connections.

**Cons**:
- May lead to rejection of requests if the client exceeds the limit.
- Doesn't handle total requests over time; focuses on simultaneous requests only.

---

### **6. IP-Based Rate Limiting**

**Design**:
- In this strategy, rate limiting is applied based on the **IP address** of the client making the requests.
- Each IP address is limited to a certain number of requests within a time window, preventing abuse from individual clients or IP addresses.

**Example**:
- Limit: 100 requests per hour per IP.
- If an IP exceeds this limit, requests from that IP are blocked until the rate limit resets.

**Pros**:
- Effective in preventing abuse from specific clients or regions.
- Can be used in combination with other rate-limiting strategies for extra protection.

**Cons**:
- Vulnerable to IP spoofing or proxies.
- Doesn't account for legitimate users behind shared IPs (e.g., users in a corporate network).

---

### **7. User-Based Rate Limiting**

**Design**:
- This strategy limits requests based on a **user identifier** (such as API keys, authentication tokens, or session IDs) instead of IP addresses.
- Useful when users authenticate before accessing the service, ensuring rate limits are applied per user rather than per IP.

**Example**:
- Limit: 1000 requests per user per day.
- If the user exceeds this limit, they are blocked from making further requests until the limit resets.

**Pros**:
- More accurate, as it targets individual users, even if multiple users are behind the same IP.
- Works well with systems that require user authentication (e.g., login systems, APIs requiring API keys).

**Cons**:
- Doesn't prevent abuse from users who generate multiple accounts or tokens.
- Requires user authentication, which may not be applicable in all cases.

---

### **8. Quota-Based Rate Limiting**

**Design**:
- This strategy involves allocating a certain **quota** (a fixed amount of allowed requests) for a client or user over a set time period.
- Once the quota is reached, the client cannot make additional requests until the quota resets, usually after a set period (e.g., daily, weekly).

**Example**:
- Limit: 500 requests per user per day.
- Once the user exceeds this quota, they cannot make further requests until the next day.

**Pros**:
- Offers flexible limits based on time or usage patterns.
- Works well for balancing traffic over extended periods.

**Cons**:
- May require more complex tracking of usage.
- Doesn't handle burst traffic well, as the user may make many requests early and then exhaust their quota.

---

### **9. Global Rate Limiting**

**Design**:
- This strategy applies rate limiting globally across all users or clients.
- The system tracks the total number of requests made across all clients and ensures the overall number of requests stays within a predefined global limit.

**Example**:
- Limit: 10,000 requests per minute globally.
- Once the total requests exceed the limit, all subsequent requests are rejected until the limit resets.

**Pros**:
- Ensures that the service doesn’t get overwhelmed by too many requests overall.
- Useful for managing total system resources.

**Cons**:
- Can lead to denial of service for individual clients during periods of heavy traffic.
- Less granular control over individual user behavior.

---

### Conclusion

The choice of **rate limiting strategy** depends on your use case, traffic patterns, and the overall architecture of your system. For example, **Token Bucket** is good for managing burst traffic, while **Fixed Window** is simpler and easier to implement for basic rate-limiting needs. 

In a **microservices architecture** with an API Gateway, combining multiple strategies—like **IP-based** with **User-based** or **Sliding Window** with **Quota-based**—might provide the most robust protection against misuse, while also offering flexibility and scalability.
