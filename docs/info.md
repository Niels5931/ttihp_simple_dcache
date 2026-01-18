## How it works

The project instantiates a `dcache_ahb_ctrl` SystemVerilog core. It is configured as a very small 8-entry cache with 8-bit words and 8-bit addressing.

### Cache Architecture
- **Direct Mapped**: Each memory address maps to a specific cache line.
- **Write-Through/Write-Allocate**: (As per the base core implementation).
- **Tagging**: Uses internal memory to store validity and address tags.

### CPU Interface
Because Tiny Tapeout has limited pins, the 8-bit data output is multiplexed. The user selects which 4-bit nibble to view using `ui_in[7]`.

### Memory Interface FSM
An internal FSM bridges the AHB-Lite signals from the cache core to the `uio` pins:
- **IDLE**: Waiting for cache miss/memory request.
- **DRIVE**: Asserts `uio_oe` and drives `8'h69` to signal a memory access cycle to external hardware.
- **SAMPLE**: Releases `uio_oe` and samples `uio_in` into the cache's AHB read data register.

## How to test

1. **Initial State**: Ensure `rst_n` is pulsed low to initialize the cache and FSM.
2. **Read Request**:
   - Set `ui_in[3:0]` to the desired address.
   - Set `ui_in[5]` to `0` (Read).
   - Pulse `ui_in[4]` (Valid) high.
   - Wait for `uo_out[5]` (Response Valid) to go high.
   - Read the data on `uo_out[3:0]`. Toggle `ui_in[7]` to see the other half of the byte.
3. **Write Request**:
   - Set address and data bit (`ui_in[6]`).
   - Set `ui_in[5]` to `1` (Write).
   - Pulse `ui_in[4]` (Valid).

## External hardware

This design can be tested with a logic analyzer or a microcontroller simulating the "Memory" on the `uio` pins. When the `uio` pins drive `0x69`, the microcontroller should respond by driving the requested data onto the same pins in the following clock cycle.