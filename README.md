# Itch-Parser
Itch Parser
ITCH market-data parser (SystemVerilog)

A SystemVerilog parser for Nasdaq ITCH-style market data. It accepts one byte per clock cycle and extracts fields from three message types: Add Order (A), Order Executed (E), and Order Delete (D).

I tested the parser in Vivado using 60 generated messages and a Python reference model. All 60 passed.

Block diagram

flowchart LR
    G[gen.py] --> M[messages.mem]
    G --> E[expected.txt]
    M --> T[SystemVerilog testbench]
    T -->|valid, sop, data| P[itch_parser]
    P -->|msg_type, order_ref, side, shares, price, out_valid| T
    T --> A[actual.txt]
    A --> C[compare.py]
    E --> C
    C --> R[PASS / FAIL]

How it works

The parser uses a byte counter to track its position within each message. When sop is asserted, the counter resets and the parser reads the message type from byte 0.

Each message type has a fixed length and field layout. The parser shifts incoming field bytes into registers in most significant byte first order. Once it receives the final byte, it asserts out_valid for one clock cycle.

Message

Length

Fields extracted

A Add Order

36 bytes

Order reference (11-18), side (19), shares (20-23), price (32-35)

E Order Executed

31 bytes

Order reference (11-18), executed shares (19-22)

D Order Delete

19 bytes

Order reference (11-18)

All offsets are zero-based.

Verification

I used a Python script to generate test messages and calculate the expected outputs.

scripts/gen.py generates 60 messages with randomly selected types and payload bytes. A fixed seed makes the tests reproducible. It writes the input stream to messages.mem and the expected fields to expected.txt.

The SystemVerilog testbench reads messages.mem and sends the messages to the parser back to back. It also inserts one cycle with valid low inside every message to test whether the byte counter handles stalls.

The testbench records the parser outputs in actual.txt.

scripts/compare.py checks actual.txt against expected.txt.

All 60 messages matched the expected outputs.

Results

Metric

Result

Messages passed

60 / 60

Output latency

1 clock cycle after the final input byte

Clock constraint

10 ns (100 MHz)

Setup slack (WNS)

2.104 ns

Hold slack (WHS)

0.054 ns

Pulse width slack (WPWS)

4.500 ns

Tool

Vivado 2026.1

Target FPGA

xc7a100tcsg324-1

The latency measurement was consistent across all 60 messages in simulation.

The timing results come from Vivado with a 10 ns clock constraint. Input and output delays are not constrained, so these results do not establish end-to-end interface timing.

Running the tests

In WSL, run python3 scripts/gen.py to generate the test data. Update the output directory in the script first.

Open the project in Vivado and run behavioral simulation. Enter run all in the Tcl console. Update the file paths in the testbench if needed.

Back in WSL, run python3 scripts/compare.py to check the results.

The comparison script prints PASS or FAIL.

Limitations

The parser accepts one byte per cycle. It does not handle wider input interfaces.

The testbench supplies sop. Ethernet decoding and packet framing are outside the project scope.

Only A, E, and D messages are supported.

Input and output delays are not constrained in the timing analysis.

The design has been simulated and analyzed in Vivado but has not been tested on physical FPGA hardware.

Repository structure

rtl/
  itch_parser.sv
tb/
  test_validation.sv
scripts/
  gen.py
  compare.py
docs/
  waveform screenshot
