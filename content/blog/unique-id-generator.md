+++
title = 'Unique ID Generator'
date = 2024-05-30T21:37:51+05:30
cover.image = "/images/unique_id.jpg"
showToc = true
+++

# System Design of a Unique ID Generator

## Requirements

Before diving into the design, let’s outline the key requirements for our unique ID generator:

1. **Uniqueness**: Every ID must be unique.
2. **Scalability**: The system should be able to handle a large number of ID generation requests.
3. **Low Latency**: ID generation should be fast.
4. **Fault Tolerance**: The system should handle failures gracefully.

## High-Level Design

We can achieve these requirements using a distributed system architecture. One popular approach is to use a combination of **Snowflake IDs** (used by Twitter) and **UUIDs**. For simplicity and efficiency, we will focus on the Snowflake ID approach.

### Snowflake ID Structure

A Snowflake ID is a 64-bit number composed of different parts:

- **Timestamp** (41 bits): Milliseconds since a custom epoch.
- **Data Center ID** (5 bits): Identifies the data center.
- **Machine ID** (5 bits): Identifies the machine within the data center.
- **Sequence Number** (12 bits): A counter for IDs generated within the same millisecond.

This structure ensures that IDs are k-ordered (roughly ordered by the time of creation).

### System Components

1. **ID Generator Service**: A service running on multiple nodes responsible for generating unique IDs.
2. **Clock Synchronization**: Ensures that the system clocks are synchronized to prevent duplicate IDs.
3. **Configuration Service**: Manages the allocation of data center IDs and machine IDs.

## Detailed Design

### ID Generator Service

Each instance of the ID generator service generates IDs based on the current timestamp, data center ID, machine ID, and sequence number. Here’s a basic implementation in Python:

```python
import time
import threading

class SnowflakeIDGenerator:
    def __init__(self, data_center_id, machine_id, epoch=1288834974657):
        self.epoch = epoch
        self.data_center_id = data_center_id
        self.machine_id = machine_id
        self.sequence = 0
        self.last_timestamp = -1

        self.data_center_id_bits = 5
        self.machine_id_bits = 5
        self.sequence_bits = 12

        self.max_data_center_id = -1 ^ (-1 << self.data_center_id_bits)
        self.max_machine_id = -1 ^ (-1 << self.machine_id_bits)
        self.max_sequence = -1 ^ (-1 << self.sequence_bits)

        self.data_center_id_shift = self.sequence_bits + self.machine_id_bits
        self.machine_id_shift = self.sequence_bits
        self.timestamp_shift = self.sequence_bits + self.machine_id_bits + self.data_center_id_bits

        self.lock = threading.Lock()

    def _timestamp(self):
        return int(time.time() * 1000)

    def _wait_for_next_millisecond(self, last_timestamp):
        timestamp = self._timestamp()
        while timestamp <= last_timestamp:
            timestamp = self._timestamp()
        return timestamp

    def generate_id(self):
        with self.lock:
            timestamp = self._timestamp()

            if timestamp < self.last_timestamp:
                raise Exception("Clock moved backwards. Refusing to generate id")

            if timestamp == self.last_timestamp:
                self.sequence = (self.sequence + 1) & self.max_sequence
                if self.sequence == 0:
                    timestamp = self._wait_for_next_millisecond(self.last_timestamp)
            else:
                self.sequence = 0

            self.last_timestamp = timestamp

            id = ((timestamp - self.epoch) << self.timestamp_shift) | \
                 (self.data_center_id << self.data_center_id_shift) | \
                 (self.machine_id << self.machine_id_shift) | \
                 self.sequence

            return id


# Instantiate SnowflakeIDGenerator with data center ID and machine ID
data_center_id = 1  # Example data center ID
machine_id = 1  # Example machine ID
generator = SnowflakeIDGenerator(data_center_id, machine_id)

# Generate a unique ID
unique_id = generator.generate_id()

# Print the generated ID
print("Generated Unique ID:", unique_id)

```
### Clock Synchronization

To avoid clock drift issues, use a service like NTP (Network Time Protocol) to keep the system clocks synchronized across all servers. This is crucial for ensuring that the timestamp component of the ID remains unique.

### Configuration Service

The configuration service is responsible for assigning unique data center IDs and machine IDs. This can be managed manually or via an automated service that ensures no two instances get the same combination of data center ID and machine ID. The configuration service should also handle the assignment of new IDs when new data centers or machines are added to the system.
### Fault Tolerance

To ensure fault tolerance:
1. Deploy multiple instances of the ID generator service across different availability zones or regions.
2. Use load balancers to distribute requests among the instances.
3. Implement health checks and failover mechanisms to handle instance failures.
4. Use a highly available and durable storage system for the configuration service to persist data center and machine ID assignments.

## Conclusion

Designing a unique ID generator requires addressing several critical requirements, including uniqueness, scalability, low latency, and fault tolerance. The Snowflake ID approach, combined with a distributed system architecture, provides an efficient and robust solution for generating unique IDs at scale.

However, it's important to note that this design assumes a relatively stable and predictable environment, where the number of data centers and machines remains within the limits of the allocated bits. If the system needs to scale beyond these limits, additional measures may be required, such as increasing the number of bits allocated for data center and machine IDs or adopting a different ID generation strategy.

Furthermore, while the Snowflake ID approach ensures rough time-ordering of IDs, it may not be suitable for applications that require strict time-ordering or have specific requirements for data partitioning or sharding based on the ID structure.

When implementing a unique ID generator, it's crucial to consider the specific requirements of the application, the expected scale, and the trade-offs involved in different design choices. Thorough testing, monitoring, and continuous optimization are essential to ensure the system's reliability and performance as it evolves over time.