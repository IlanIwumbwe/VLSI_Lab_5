# Scan chain insertion, LEC, power estimation

## Scan chain insertion
First we synthesize the module, then save the design for future use using:
```
write_design -base_name ${_OUTPUTS_PATH}/DESIGN/${DESIGN}_synth
```

### Inserting scan chains
We configure the DFT scan style

`scan_en`: Enables scan mode by selecting the scan input via the multiplexer.
`scan_testmode`: Forces the design into test mode, disabling or overriding parts of the circuit for testing.

`convert_to_scan` command reports percentage of registers available for DFT: 
```
Scan mapping status report
==========================
    Scan mapping: converting flip-flops that pass TDRC.
      Scan connection mode: 'loopback'.
      Scan shift-enable connection mode: 'tie_off'.
    Scan mapping done: 525 flip-flops mapped to scan.
    Category                               Number    Percentage
    -----------------------------------------------------------
    Scan flip-flops mapped for DFT           525        100.00%
    Flip-flops not mapped for DFT
         flip-flops not scan replaceable       0          0.00%
         flip-flops not targeted for DFT       0          0.00%
    -----------------------------------------------------------
                                  Totals     525        100.00%
```

### Inspecting the results

Original interface
```sv
  3 module FIFO #(
  4   parameter int WIDTH = 32,
  5   parameter int DEPTH = 16
  6 )(
  7     input  logic              clk,
  8     input  logic              rst_n,
  9
 10     input  logic              clk_en,
 11
 12     input  logic              in_valid,
 13     output logic              in_ready,
 14     input  logic [WIDTH-1:0]  in_data,
 15
 16     output logic              out_valid,
 17     input  logic              out_ready,
 18     output logic [WIDTH-1:0]  out_data
 19 );
```
**Identify new ports**

Timing before scan chain insertion:
```sv
 18                      Capture       Launch
 19         Clock Edge:+    1100            0
 20        Src Latency:+       0            0
 21        Net Latency:+       0 (I)        0 (I)
 22            Arrival:=    1100            0
 23
 24              Setup:-      56
 25        Uncertainty:-      22
 26      Required Time:=    1022
 27       Launch Clock:-       0
 28          Data Path:-    1022
 29              Slack:=       0
```

Timing after scan chain insertion:
```sv
 11 Path 1: VIOLATED (-128 ps) Setup Check with Pin mem_reg[1][14]/CP->D
 12           Group: clk
 13      Startpoint: (R) count_reg[4]/CP
 14           Clock: (R) clk
 15        Endpoint: (F) mem_reg[1][14]/D
 16           Clock: (R) clk
 17 
 18                      Capture       Launch
 19         Clock Edge:+    1100            0
 20        Src Latency:+       0            0
 21        Net Latency:+       0 (I)        0 (I)
 22            Arrival:=    1100            0
 23 
 24              Setup:-     198
 25        Uncertainty:-      22
 26      Required Time:=     880
 27       Launch Clock:-       0
 28          Data Path:-    1008
 29              Slack:=    -128
```
We see a difference in the slack time, with that after scan chain insertion, getting negative slack time. The data path is also slightly lower.

**Why?**

Area before insertion:
```
 13 Instance Module  Cell Count  Cell Area  Net Area   Total Area
 14 --------------------------------------------------------------
 15 FIFO                   1847   5965.680  2473.033     8438.713
```

Area after insertion:
```
 13 Instance Module  Cell Count  Cell Area  Net Area   Total Area
 14 --------------------------------------------------------------
 15 FIFO                   1875   7662.480  3102.973    10765.453
```
Adding scan chains increases the area taken up by the design

Power consumption before:
```
 10                  Leakage    Dynamic      Total
 11 Instance  Cells Power(nW)  Power(nW)   Power(nW)
 12 --------------------------------------------------
 13 FIFO       1847  1395.908 7543549.599 7544945.506
```

Power consumption after:
```
 10                  Leakage    Dynamic      Total
 11 Instance  Cells Power(nW)  Power(nW)   Power(nW)
 12 --------------------------------------------------
 13 FIFO       1875  1774.808 8704755.677 8706530.485
```
We consume more power after scan insertion because the design now takes up more area, hence use up more transistors, which increases leakage power.

## LEC
Reloaded post-DFT design, genus then generates a script to do the LEC. Running this command yields:

```
--------------------------------------------------------------------------------
6. Compare Results:                                                        PASS
     Number of EQ compare points:                              559
     Number of NON-EQ compare points:                          0
     Number of Aborted compare points:                         0
     Number of Uncompared compare points :                     0
================================================================================
```





