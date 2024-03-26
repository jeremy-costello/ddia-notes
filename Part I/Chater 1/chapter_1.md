# Chapter 1: Reliable, Scalable, and Maintainable Applications

Many modern applications are data-intensive rather than compute-intensive.

Some building blocks for data-intensive applications:
1. Databases - store data for later retrieval
2. Caches - remember results of expensive operations
3. Search indexes - search and filter data
4. Stream processing - send asynchronous messages between processes
5. Batch processing - periodically process accumulated data

There are various approaches to all of these, with multiple available tools for each approach.

Applications often require multiple of these building blocks, so they are stitched together using application code.

Three important concerns:
1. Reliability - system works correctly through adversity
2. Scalability - system can deal with growth
3. Maintainability - easy to maintain current behaviour and adapt to new use cases

#### Reliability
- Error handling
- Purposely trigger faults
    - e.g. Netflix's chaos monkey
    - [Principles of Chaos Engineering](https://principlesofchaos.org/)
- Hardware faults
    - Redundancy: RAID, dual power supplies, hot-swappable CPUs, backup power
    - Move towards tolerating the loss of entire machines
- Software errors
    - Hardware faults are usually uncorrelated. Software errors can be correlated
    - Solutions: careful forethought, process isolation, allowing crashes, monitoring system behaviour
- Human errors
    - Minimize opportunities for error, sandbox environments for experimenting, thorough automated testing, quick and easy recovery, detailed and clear monitoring, good management practices and training

#### Scalability
- A system's ability to cope with increased load
- Describing load
    - Requests per second, read/write ratio, active users
    - Twitter example: fanning out tweets to followers (distribution of followers per user)
- Describing performance
    - How is performance affected upon increasing load with constant resources?
    - Upon increasing load, what resource increase is required for constant performance?
    - Thoroughput, response time (average, median, percentiles)
    - Choose a target response time for various percentiles
    - Algorithms for approximating percentiles: forward decay, t-digest, HdrHistogram
    - Add histograms, don't average percentiles
- Approaches for coping with load
    - vertical (more powerful machine) versus horizontal (more machines) scaling
    - elastic systems (highly unpredictable loads) versus manually scaled systems
    - stateless services are easy to distribute, while stateful services are not
    - read volume, write volume, data volume, data complexity, response time, access patterns
    - assumptions of which operations will be common and which will be rare
    - application architectures are unique, but built from familiar blocks

#### Maintainability
- Three design principles
    - operability: easy to keep the system running smoothly
    - simplicity: easy to understand the system
    - evolvability: easy to make changes to the system
- Operability
    - operations teams are vital to keeping a software system running smoothly
    - make routine tasks easy, allowing the operations team to focus on high-value activities
- Simplicity
    - projects become more complex and difficult to understand as they get larger
    - good abstractions help a lot with simplicity; extracting parts of a large system into well-defined, reusable components
- Evolvability
    - agile working patterns provide a framework for adapting to change
    - simple and easy-to-understand systems are usually easier to modify than complex ones
