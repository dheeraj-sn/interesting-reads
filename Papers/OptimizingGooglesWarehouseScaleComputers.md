# [Optimizing Google's Warehouse Scale Computers: The NUMA Experience](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/40691.pdf)

## Summary

The central finding is that Non-Uniform Memory Access (NUMA) has a significant performance impact on Google's large-scale, real-world applications. Optimizing for memory locality is not a minor tweak but a critical factor for efficiency.

### Key Conclusions

1. **Quantifiable Performance Gains**: By improving memory locality (i.e., ensuring an application's code running on a CPU accesses memory attached to that same CPU), performance can be improved by 10-20% for major services like the Gmail backend and web-search frontend.

2. **The "More Local is Better" Rule Isn't Always True**: The most surprising conclusion is that the simplest optimization strategy—always maximizing local memory access—can sometimes hurt performance. There is a crucial and complex tradeoff between NUMA locality and cache contention.

3. **The NUMA vs. Cache Tradeoff**:
   - Clustering an application's threads onto a single processor socket improves memory locality but can increase competition for that socket's shared cache.
   - Spreading threads across multiple sockets reduces cache contention but increases the number of slow, remote memory accesses.
   - In some load-tests, applications performed better with 100% remote memory access than with 50% local access, because avoiding cache contention was more important than the penalty of remote memory access.

4. **Methodology is Key**: A two-phase approach is essential for understanding these issues in a massive, complex environment:
   - First, use large-scale production monitoring to identify performance opportunities (like the general impact of NUMA)
   - Second, use controlled, single-node load-testing to diagnose the complex interactions and root causes (like the cache contention tradeoff)

---

## Analysis: "Dedicated CPU is a Lie: The Hidden Hardware Costs our Backend is Paying"

> As backend engineers, we're trained to obsess over algorithmic complexity, database queries, and network latency. But what if the biggest performance killer isn't in our code, but in the silicon it runs on?

The way we think about server hardware is often a dangerous oversimplification. We treat a server as a flat pool of resources—X cores and Y RAM—but the reality is far more complex and has profound implications for our applications.

**The most jarring takeaway?** Our application's performance can be wrecked by a "noisy neighbor" process, even if we have dedicated CPU cores.

### The Myth of the Isolated Core

You deploy your shiny new microservice to a container with a request for 4 CPU cores. You assume you have a private, isolated fortress. The truth is, your cores live in a neighborhood and have to share common infrastructure. The two most critical shared resources are:

#### 1. The L3 Cache (The Shared Whiteboard)
All cores on a single CPU socket share a large L3 cache. Think of it as a big whiteboard that all your cores use for quick notes. Now, imagine a "noisy neighbor" application—like a data analytics job—starts running. It's like a coworker who erases all your notes to fill the entire whiteboard with their own work. The next time your app needs its data, it's gone from the cache (a "cache miss"), and it has to make a slow trip to main memory. This is **cache pollution**, and it tanks performance.

#### 2. The Memory Controller (The Warehouse Door)
All cores on a socket also share a memory controller to access the main RAM. If a neighbor app is memory-intensive, it can saturate the controller, creating a traffic jam. Your app's small, quick requests get stuck in the same line as the neighbor's huge, slow ones.

### The NUMA Dilemma

Modern servers are NUMA (Non-Uniform Memory Access) systems. This means a CPU socket is physically closer to some banks of RAM than others. Accessing its "local" memory is fast. Accessing "remote" memory attached to another CPU socket is significantly slower.

This creates a vicious trade-off:

- **Option A: Pin to one socket** - You can schedule all your app's threads on one socket to ensure 100% fast, local memory access. The risk? All your threads are now fighting over the same, smaller L3 cache—the "shared whiteboard" problem gets worse.

- **Option B: Spread across sockets** - You can spread your threads across multiple sockets to give them more cache space. The risk? Now your threads are constantly making slow, remote memory accesses to get data from other sockets.

The Google paper showed that, counter-intuitively, sometimes **Option B is better!** The performance penalty from cache contention can be worse than the penalty from remote memory access.

### What Can We Do About It? 3 Actionable Steps

This isn't just an academic problem. It explains those maddening "Heisenbugs" where performance tanks for no reason. Here's how we can fight back:

#### 1. Become "Topology-Aware"
Stop thinking of servers as flat. On any Linux machine, run `lscpu`. The output will show you the NUMA node architecture. Understanding the physical layout of the machine your code runs on is the first step.

#### 2. Profile Smarter
Your standard CPU % and memory usage won't catch this. You need to look deeper. Metrics like **CPI (Cycles Per Instruction)** and **L3 Cache Miss Rate** are your best friends. A sudden spike in these metrics without a change in your app's load is a huge red flag for a noisy neighbor problem.

#### 3. Ask the Right Questions
This knowledge empowers you to have better conversations with your SRE and Platform teams:
- "Is our Kubernetes cluster using a NUMA-aware TopologyManager policy?"
- "Can we schedule our latency-critical services on nodes that are isolated from batch-processing workloads?"
- "For our most critical service, can we pin it to a specific NUMA node using `numactl` to guarantee predictable performance?"

---

**The next frontier of performance optimization isn't just in our code; it's in understanding the complex hardware landscape it operates in. By moving beyond the myth of the flat server, we can build faster, more reliable, and more resilient systems.**