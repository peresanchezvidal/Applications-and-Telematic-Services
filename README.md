# Applications & Telematic Services — Laboratory Practices (README)

## Description
This repository holds the laboratory practice code and minimal tests for the course *Applications and Telematics Services*.  
Its purpose is to provide clean, instructor-aligned implementations of transport-layer concepts (queues, simulated network, fragmentation, ARQ, multiplexing, sliding windows, connection state machines, concurrency) together with the small test drivers used in the handout so you can reproduce the expected console traces. The code is organized per practice (practica1..practica7). Each practice implements a focused concept and contains: source (`src/`), small test mains (`test/`), and a short `README.md` explaining how to run the provided examples.

## Contents (detailed)
- `practica1/` — **Data structures & basic transport socket**
  - Goal: implement and validate the fundamental data containers and a minimal send/receive socket over the simulated network.
  - Key classes: `CircularQueue<E>.java`, `LinkedQueue.java`, `SimNet_Queue.java`, `TSocketSend.java`, `TSocketRecv.java`.
  - Tests: small mains such as `TestCQ.java`, `TestLQ.java` that exercise enqueue/dequeue, and `TestTSocketBasic.java` that sends a short payload and expects simple `PSH`/`ACK` trace lines.
  - Expected behaviors: correct FIFO ordering, wrap-around behavior for circular queue, correct segmentation by TSocket when message fits within MSS, and matching println traces used by graders.

- `practica2/` — **Concurrency and monitors**
  - Goal: convert single-threaded network and protocol code to correct multi-threaded versions using monitors/locks.
  - Key classes: `SimNet_Monitor.java`, `MonitorCZ.java`, `SenderThread.java`, `ReceiverThread.java`, `ProtocolMonitor.java`.
  - Tests: threaded scenarios that stress producer/consumer behavior and require using `wait()`/`notify()` (or Java `Condition`) properly.
  - Expected behaviors: no deadlocks, no lost wakeups, stable ordering when multiple threads access the simulated network, and exact printed traces when interleavings are fixed.

- `practica3/` — **Fragmentation & lossy network**
  - Goal: implement segmentation at sender, reassembly at receiver, and handle MTU/MSS constraints and simulated packet loss.
  - Key classes: `TSocketSend.segmentize(...)`, `TSocketRecv.processReceivedSegment(...)`, `SimNet_Loss.java`.
  - Tests: fragment a message longer than MSS, simulate loss of selected segments and verify retransmission behavior in later practices, and reassembly producing original payload.
  - Expected behaviors: segments labeled with offsets/IDs, receiver buffers out-of-order segments until contiguous data can be delivered, and printed logs for `FRAG`/`REASS` events.

- `practica4/` — **Multiplexing & demultiplexing**
  - Goal: support multiple logical sockets on the same simulated network interface and route incoming segments to the correct endpoint.
  - Key classes: `Protocol.java` (ipInput / demux table), `TSocket.java` (socket identifier, bind/connect).
  - Tests: two concurrent sockets exchanging data while the protocol demultiplexes; validate port/socket id matching.
  - Expected behaviors: `ipInput` routes segments by `(srcAddr, srcPort, dstAddr, dstPort)`, and trace lines show proper delivery to the right `TSocketRecv`.

- `practica5/` — **Stop-and-Wait ARQ**
  - Goal: make a simple reliable transfer using stop-and-wait with ACK/NACK handling and timeout-based retransmission.
  - Key classes: `ARQStopAndWait.java`, `TimerTask` hooks within `TSocketSend`, `TSocketRecv` ack handling.
  - Tests: single-packet send with deliberately dropped ACK to force one retransmission; expected two `snd -> [PSH ...]` traces and a final `received: N bytes`.
  - Expected behaviors: correct sequence numbering (0/1), retransmit on timeout, and idempotent receiver (duplicate detection).

- `practica6/` — **Sliding window & pipelining**
  - Goal: implement a windowed sender (N>1), cumulative ACKs, out-of-order buffering, and retransmission on timeout.
  - Key classes: `SlidingWindowSender.java`, `ReceiverWindowBuffer.java`, congestion-window adjustments if required.
  - Tests: send a multi-segment transfer with selective losses causing retransmissions and out-of-order arrival; validate effective throughput traces and correct final payload delivery.
  - Expected behaviors: sender maintains `sendBase` and `nextSeq`, retransmits only timed-out segments, receiver buffers out-of-order segments and delivers in-order data to the application.

- `practica7/` — **Connection setup & teardown (TCP-like FSM)**
  - Goal: model a simple three-way handshake and orderly teardown (SYN, SYN-ACK, ACK, FIN, FIN-ACK).
  - Key classes: `ConnectionStateMachine.java`, `TSocketConnect.java`.
  - Tests: establish a connection, exchange a small payload, close connection and verify proper state transitions and logs.
  - Expected behaviors: transitions follow the handout state diagram, no data sent on closed sockets, and connection logs match expected sequence (SYN→SYN-ACK→ACK, then FIN/FIN-ACK).

---

Each practice folder should include a minimal `README.md` that lists the exact `javac` and `java` commands to compile and run the test mains for that practice and the names of the classes implemented.
