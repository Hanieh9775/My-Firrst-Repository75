"""
Token Bucket Rate Limiter
-------------------------
A production-style rate limiter implementation.

Concepts:
- Token Bucket Algorithm
- Thread safety
- Time-based refill
- Used in APIs, gateways, distributed systems

Run:
python rate_limiter.py
"""

import time
import threading


class TokenBucket:
    def __init__(self, capacity: int, refill_rate: float):
        """
        capacity: maximum number of tokens
        refill_rate: tokens added per second
        """
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = capacity
        self.last_refill = time.monotonic()
        self.lock = threading.Lock()

    def _refill(self):
        now = time.monotonic()
        elapsed = now - self.last_refill
        added_tokens = elapsed * self.refill_rate

        if added_tokens > 0:
            self.tokens = min(self.capacity, self.tokens + added_tokens)
            self.last_refill = now

    def allow_request(self, tokens: int = 1) -> bool:
        with self.lock:
            self._refill()

            if self.tokens >= tokens:
                self.tokens -= tokens
                return True

            return False


def simulate_requests(bucket: TokenBucket, total_requests: int, interval: float):
    allowed = 0
    blocked = 0

    for _ in range(total_requests):
        if bucket.allow_request():
            allowed += 1
            print("Request allowed")
        else:
            blocked += 1
            print("Request blocked")

        time.sleep(interval)

    print("\nSimulation result")
    print("-----------------")
    print(f"Allowed requests: {allowed}")
    print(f"Blocked requests: {blocked}")


def main():
    # 5 requests per second, burst up to 10
    bucket = TokenBucket(capacity=10, refill_rate=5)

    simulate_requests(
        bucket=bucket,
        total_requests=30,
        interval=0.1
    )


if __name__ == "__main__":
    main()
