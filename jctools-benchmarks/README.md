JCTools Benchmarks
-------
This project contains performance tests designed to stretch the queue implementations and expose contention as well
as providing some baseline quantities to consider when doing back of the envelope estimations. The benchmarks cover
some basic operation costs, latency (for a SPSC use case) and throughput.

Building the benchmarks
-------
The benchmarks are maven built and involve some code generation for the JMH part. As such it is required that you
run 'mvn clean install' on changing the code. As the codebase is rather small it is recommended that you run this
command from the parent folder to avoid missed changes from other packages.

Running the benchmarks: General
-------
It is recommended that you consider some basic benchmarking practices before running benchmarks:

 1. Use a quiet machine with enough CPUs to run the number of threads you mean to run.
 2. Set the CPU freq to avoid variance due to turbo boost/heating.
 3. Use an OS tool such as taskset to pin the threads in the topology you mean to measure.

The benchmarks included are both JMH benchmarks(under org.jctools.jmh) and some handrolled benchmarks (under
org.jctools.handrolled). The JMH benchmarks pick their queue with a JMH parameter, 'qType', which takes a class name.
The handrolled and latency benchmarks still use the 'q.type' system property, which takes a number defined in the
QueueByTypeFactory class.
Note that all SPSC benchmarks can be used to test MPMC/SPMC/MPSC queues as they cover a particular case for those.

Running the JMH Benchmarks
-----
To run all JMH benchmarks:

    java -jar target/microbenchmarks.jar -f <number-of-forks> -wi <number-of-warmup-iterations> -i <number-of-iterations>
To list available benchmarks:

    java -jar target/microbenchmarks.jar -l
Some JMH help:

    java -jar target/microbenchmarks.jar -h
Example:

To run the throughput benchmark for the MpmcArrayQueue:

    java -jar target/microbenchmarks.jar ".*.QueueThroughput.*" -p qType=MpmcArrayQueue

The 'qType' parameter is a simple class name, resolved against org.jctools.queues, org.jctools.queues.varhandle,
org.jctools.queues.atomic, java.util and java.util.concurrent - so 'MpscUnpaddedArrayQueue', 'ArrayBlockingQueue' and
'ConcurrentLinkedQueue' all work too, which is handy for comparing against the JDK. Capacity is set with 'qCapacity',
and the linked array queues take a 'chunk.capacity' form:

    java -jar target/microbenchmarks.jar ".*.QueueThroughput.*" -p qType=MpscChunkedArrayQueue -p qCapacity=1024.132000

This particular benchmark allows the testing of multiple consumers/producer threads by using thread groups:

    java -jar target/microbenchmarks.jar ".*.QueueThroughput.*" -p qType=MpmcArrayQueue -tg 4,4

The tg option will set 4 consumers and 4 producers to the benchmark. You can play with the other options as described
in the JMH help.

Note that '-Dq.type' has no effect on these benchmarks - it is only read by the handrolled benchmarks and the SPSC
latency benchmarks.

Running the handrolled benchmarks
-----
The handrolled benchmarks are currently only covering SPSC throughput. These can be run by directly invoking the class:

    java -Dq.type=7 -cp target/microbenchmarks.jar org.jctools.handrolled.throughput.spsc.QueuePerfTest

Here 'q.type' is the queue type number from QueueByTypeFactory (7 being the MpmcArrayQueue).


