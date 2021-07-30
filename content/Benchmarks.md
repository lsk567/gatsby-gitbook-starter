This page collects data on benchmark measurements of Lingua Franca programs. The hardware on which these tests are run is detailed [below](#Hardware).

## Internal Benchmarks

These are programs that are not part of any standard benchmark suite, but which we use to measure the impact of design changes.

* **test/C/TimeLimit.lf**: This program sends a counting sequence of length 10,000,002 from a source to a destination. Is is meant to measure the basic overhead of reaction invocation.<br/>
  **Mac1**: 583 msec (AVG), 562 msec (MIN), 648 msec (MAX), 577 msec (MED) (100 runs - Oct. 14, 2020)<br/>
  **AGX**: 1034 msec (AVG), 1031 msec (MIN), 1135 msec (MAX), 1031 msec (MED) (100 runs - Oct. 14, 2020)<br/>
  **Wessel**: 793 msec (AVG), 782 msec (MIN), 847 msec (MAX), 786 msec (MED) (100 runs - Dec. 5, 2020)

* **test/C/TimeLimitThreaded.lf**: This version of the previous test uses the threaded runtime. There is no exploitable parallelism, so no speedup is expected.<br/>
  **Mac1** (with threads = 8): 1250 msec (MIN), 1088 msec (MIN), 1339 msec (MAX), 1308 msec (MED) (100 runs - Oct. 14, 2020)<br/>
  **AGX** (with threads = 8): 1660 msec (AVG), 1650 msec (MIN), 1926 msec (MAX), 1656 msec (MED) (100 runs - Oct. 15 2020)<br/>
  **Wessel** (with threads = 8): 1213 msec (AVG), 1195 msec (MIN), 1605 msec (MAX), 1212 msec (MED) (100 runs - Oct. 15 2020)
## Savina Benchmarks

The [Savina benchmark suite](https://github.com/shamsmahmood/savina) for actor languages and frameworks provides a number of useful patterns for measuring performance of Lingua Franca programs. See the [paper on Savina](https://shamsimam.github.io/papers/2014-agere-savina.pdf). None of these benchmarks has any notion of time, and not all comparisons with the actor implementations will be fair.

* **benchmark/C/Savina/PingPong.lf**: This benchmark tests a feedback interaction where `Ping` sends a message to `Pong`, which responds, triggering `Ping` to send another message. This gets repeated 1,000,000 times.  This version of the benchmark runs in the unthreaded runtime. Based on [https://www.scala-lang.org/old/node/54].<br/>
  **Mac1**: 63 msec (AVG), 61 msec (MIN), 76 msec (MAX),62 msec (MED) (100 runs - Oct. 17, 2020)<br/>
  **AGX**: 114 msec (AVG), 113 msec (MIN), 118 msec (MAX), 113 msec (MED) (100 runs - Oct. 14 2020)
* **benchmark/C/Savina/PingPongThreaded.lf**: Version of the previous benchmark runs in the threaded runtime with 8 worker threads. There is no parallelism, so no speedup is expected.<br/>
  **Mac1**: 147 msec (AVG), 144 msec (MIN), 164 msec (MAX), 145 msec (MED) (100 runs - Oct. 17, 2020)<br/>
  **AGX**: 217 msec (AVG), 216 msec (MIN), 254 msec (MAX), 216 msec (MED) (100 runs - Oct. 14 2020)
* **benchmark/C/Savina/PingPongMultiThreaded.lf**: Version of the previous benchmark runs in the threaded runtime with 8 worker threads and has two independent Ping-Pong benchmarks running together. There is parallelism, so speedup is expected, but instead, we get dramatic slowdown!<br/>
  **Mac1**: 4269 msec (AVG), 4051 msec (MIN), 4405 msec (MAX), 4375 msec (MED) (10 runs, Oct. 17, 2020)<br/>
  **AGX**: Crashed

## Hardware

**Mac1**: 
    Model Name:	MacBook Pro<br/>
    Model Identifier:	MacBookPro15,1<br/>
    Processor Name:	Intel Core i9<br/>
    Processor Speed:	2.3 GHz<br/>
    Number of Processors:	1<br/>
    Total Number of Cores:	8<br/>
    L2 Cache (per Core):	256 KB<br/>
    L3 Cache:	16 MB<br/>
    Hyper-Threading Technology:	Enabled<br/>
    Memory:	32 GB<br/>
    Operating System: macOS 10.14.6 (Mojave)<br/>

**AGX**: 
    Model Name:	NVIDIA Jetson AGX Xavier<br/>
    Model Identifier:	Jetson-AGX<br/>
    Processor Name:	ARMv8 Processor rev 0 (v8l)<br/>
    Max Processor Speed:	2265.6001 MHz (fixed for this benchmark)<br/>
    Number of Processors:	1<br/>
    Total Number of Cores:	8<br/>
    L2 Cache (per Core):	2048 KB<br/>
    L3 Cache:	4096 KB<br/>
    Hyper-Threading Technology:	N/A<br/>
    Memory:	16 GB<br/>
    Operating System: Ubuntu 18.04 (Bionic) - JetPack 4.3<br/>

**Wessel**: 
    Model Name:	Dell PowerEdge R730<br/>
    Model Identifier:	PowerEdge R730<br/>
    Processor Name:	[Intel(R) Xeon(R) CPU E5-2643 v3 @ 3.40GHz](https://ark.intel.com/content/www/us/en/ark/products/81900/intel-xeon-processor-e5-2643-v3-20m-cache-3-40-ghz.html)<br/>
    Max Processor Speed:	3.7Ghz (turbo)<br/>
    Number of Processors:	1<br/>
    Total Number of Cores:	6<br/>
    Cache: 20 MB Intel® Smart Cache<br/>
    Hyper-Threading Technology:	Yes<br/>
    Memory:	94.3 GB<br/>
    Operating System: ArchLinux<br/>