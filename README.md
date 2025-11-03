# 🧪 Experiment 5: Design and Verification of Register Memory Access using Mailbox-based Producer-Consumer Model

---

## 🎯 Aim  
To design and verify a **Register Memory Access system** using a **mailbox-based Producer-Consumer model** in **SystemVerilog**, demonstrating inter-process communication and synchronization in a concurrent environment.

---

## 🧰 Apparatus Required  
- Computer with **Windows/Linux OS**  
- **EDA Playground** (SystemVerilog environment) or **ModelSim 2020.1**  
- Internet browser (for EDA Playground execution)  

---

## 📘 Description  
This experiment demonstrates **inter-process synchronization and data transfer** using the **mailbox mechanism** in SystemVerilog.  

- The **Producer process** writes data into a **mailbox**, simulating register write access.  
- The **Consumer process** retrieves data from the mailbox, representing register read access.  
- Mailboxes allow safe **communication between concurrent processes** without data corruption.  
- Verification ensures that the written data matches the read data (register consistency).  

---

## ⚙️ Features  
- Demonstrates **Producer–Consumer synchronization** using mailbox  
- Implements **Register Memory Access** (write and read)  
- Uses **concurrent processes** (`fork...join`) in SystemVerilog  
- Self-checking **testbench verification**  

---

## 🧩 Procedure  

1. **Open EDA Playground**  
   - Go to [https://www.edaplayground.com](https://www.edaplayground.com)  
   - Select **SystemVerilog (IEEE 1800-2012)** and choose **ModelSim/QuestaSim** as the simulator.  

2. **Create Files**  
   - Create `register_memory.sv` → Design file  
   - Create `register_memory_tb.sv` → Testbench file  

3. **Paste the Code**  
   - Copy the design and testbench code from below.  

4. **Run the Simulation**  
   - Click **Run** → **Run Simulation**.  

5. **Observe Output**  
   - View the simulation log for producer/consumer transactions.  
   - Check the mailbox synchronization and matching register values.  

6. **Analyze Results**  
   - Confirm that all produced data is consumed correctly.  
   - Ensure no data mismatch or deadlock occurs.  

---

## 💻 SystemVerilog Code

### 🧱 Design File — `register_memory.sv`
```systemverilog
//=====================================================
// Design: Register Memory Model
//=====================================================
module register_memory #(parameter WIDTH = 8, DEPTH = 8) ();

    // Register memory declaration
    logic [WIDTH-1:0] mem [0:DEPTH-1];

    // Write task
    task automatic write_reg(input int addr, input logic [WIDTH-1:0] data);
        if (addr < DEPTH) begin
            mem[addr] = data;
            $display("[%0t] WRITE: Address=%0d Data=%0h", $time, addr, data);
        end else begin
            $display("[%0t] WRITE ERROR: Invalid Address %0d", $time, addr);
        end
    endtask

    // Read task
    task automatic read_reg(input int addr, output logic [WIDTH-1:0] data);
        if (addr < DEPTH) begin
            data = mem[addr];
            $display("[%0t] READ:  Address=%0d Data=%0h", $time, addr, data);
        end else begin
            $display("[%0t] READ ERROR: Invalid Address %0d", $time, addr);
        end
    endtask
endmodule
```
### Testbench File
```
//=====================================================
// Testbench: Producer-Consumer using Mailbox
//=====================================================
module register_memory_tb;

    // Parameter definitions
    parameter WIDTH = 8;
    parameter DEPTH = 8;

    // Mailbox declaration
    mailbox mbx = new();

    // Instantiate Register Memory
    register_memory #(WIDTH, DEPTH) regmem();

    // Transaction class for data transfer
    class transaction;
        rand int addr;
        rand bit [WIDTH-1:0] data;
    endclass

    transaction tx;

    // Producer Process: Generates random data and sends to mailbox
    task producer();
        repeat (5) begin
            tx = new();
            assert (tx.randomize() with {addr < DEPTH;}) else $error("Randomization failed!");
            $display("[%0t] PRODUCER: Sending (Addr=%0d, Data=%0h)", $time, tx.addr, tx.data);
            mbx.put(tx); // send transaction
            regmem.write_reg(tx.addr, tx.data);
            #10;
        end
    endtask

    // Consumer Process: Reads data from mailbox and verifies register
    task consumer();
        transaction rx;
        logic [WIDTH-1:0] read_data;
        forever begin
            mbx.get(rx); // receive transaction
            regmem.read_reg(rx.addr, read_data);
            if (read_data == rx.data)
                $display("[%0t] CONSUMER: Data Matched at Addr=%0d ✅", $time, rx.addr);
            else
                $display("[%0t] CONSUMER: Data Mismatch at Addr=%0d ❌", $time, rx.addr);
            #10;
        end
    endtask

    // Run both producer and consumer concurrently
    initial begin
        $display("=== Register Memory Access using Mailbox-Based Producer-Consumer ===");
        fork
            producer();
            consumer();
        join_any
        #50 $finish;
    end

endmodule
```
### Simulation Output




### Result

The Register Memory Access was successfully designed and verified using the mailbox-based Producer-Consumer model in SystemVerilog.
The producer wrote random data into memory, and the consumer read and validated it, demonstrating correct mailbox synchronization and data integrity.


