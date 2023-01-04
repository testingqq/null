 The Processor


                                                                     Computer



                        4
                                                                     Devices
  Chapter

  Part          I                                                    Memory


                                                                     Processor




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.     Chapter 4.Part I   1 of 69
 Agenda and Reading List
   •   Chapter goals
   •   Introduction (4.1, pp. 300-303)
   •   ISA summary
   •   The five classic components of a computer (1.4, pp. 16-17)
   •   The two classes of computer architectures
   •   Generic implementation (4.1, pp. 301-303)
   •   Clocking methodology (4.2, pp. 303-307)
   •   Datapath elements and building a datapath (4.3, pp. 307-313)
   •   Creating a single datapath (4.3, pp. 313-315)
   •   A simple implementation scheme (4.4, pp. 316-330)
   •   The ALU control (4.4, pp. 316-318)
   •   The main control unit (4.4, pp. 318-321)
   •   The operation of the datapath (4.4, pp. 321-327)
   •   Implementing unconditional jumps (4.4, p. 328)
   •   Inefficiencies in the single-cycle implementation (4.4, pp. 328-330)
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   2 of 69
 Chapter Goals
   •   to explain the principles and techniques used in implementing a
       modern RISC processor, starting with a highly abstract and
       simplified overview

   •   to develop an understanding of combinational and clocked
       sequential circuits and the relationship between them

   •   to see how the ISA determines many aspects of the implementation

   •   to construct the datapath, ALU control unit, and main control unit
       for a single-cycle implementation of the MIPS instruction set

   •   to discuss how the choice of various implementation strategies
       affects the clock rate and CPI for the computer

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   3 of 69
 Introduction
  •   The implementation of the processor determines both the clock cycle
      time and the number of clock cycles per instruction (CPI)
  •   We will be examining an implementation that includes a subset of the
      core MIPS instruction set:
       – The memory-reference instructions: lw, sw
       – The arithmetic-logical instructions: add, sub, and, or, slt
       – The control flow (branch and jump) instructions: beq, j
  •   A simple implementation that uses a single long clock cycle for
      every instruction will be shown
       – Every instruction begins execution on one clock edge and
          completes execution on the next clock edge
       – While easier to understand, this approach is not practical
           • since the clock cycle must be stretched to accommodate the
              longest instruction
           • it leads to a slow implementation
  •   We will concentrate on designing the control
       – Control is the most challenging aspect of the processor design
       – It is the hardest part to get right and to make fast
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   4 of 69
 ISA Summary
  Field bit positions 31-26 25-21 20-16                         15-11            10-6       5-0
             No. of bits          6          5          5            5             5         6
               R-format          op          rs         rt           rd         shamt       funct
                I-format         op          rs         rt      16- bit immediate/address
                J-format         op                          26-bit address

  •   The opcode field, op[5:0] is always contained in bits 31:26
  •   The two registers to be read are always specified by the rs and rt fields
      at positions 25:21 and 20:16
  •   The base register for lw and sw is always in bit positions 25:21 (rs)
  •   The 16-bit offset for lw, sw, and beq is always in bit positions 15:0
  •   The destination register is either
       – in bit positions 20:16 (rt) for lw or
       – in bit positions 15:11 (rd) for an R-format instruction
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.       Chapter 4.Part I    5 of 69
 ISA Summary (cont.)
 Instruction      Format        op      funct     Instruction        Format       op     funct
     add             R          010      3210          or              R          0       37
     addi            I           8                     ori             I         13
     and             R           0        36           sb              I         40
     andi            I          12                     sh              I         41
     beq             I           4                     sll             R          0       0
     bne             I           5                     slt             R          0       42
        j            J           2                    slti             I         10
      jal            J           3                    sra              R          0       3
       Jr            R           0         8           srl             R          0       2
      Lb             I          32                    sub              R          0       34
      Lh             I          33                    sw               I         43
      Lui            I          15                    xor              R          0       38
      Lw             I          35                    xori             I         14
     Nor             R           0        39

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.    Chapter 4.Part I    6 of 69
 ISA Summary (cont.)
  • MIPS Registers
            Register number                                                       Preserve on
  Name                                        Usage
               (decimal)                                                             call?
 $zero             0        the constant value 0                                      n.a.
 $at                 1            reserved for the assembler                            n.a.
                                  procedure return values and expression
 $v0-$v1            2-3                                                                 no
                                  evaluation
 $a0-$a3            4-7           procedure arguments (parameters)                      no
 $t0-$t7            8-15          temporary registers                                   no
 $s0-$s7           16-23          general purpose saved registers                       yes
 $t8-$t9           24-25          more temporary registers                              no
 $k0-$k1           26-27          reserved for the OS                                   n.a.
 $gp                 28           global pointer                                        yes
 $sp                 29           stack pointer                                         yes
 $fp                 30           frame pointer                                         yes
 $ra                 31           procedure return address                              yes

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I          7 of 69
 ISA Summary (cont.)
• MIPS Addressing Modes
     – Immediate addressing: the operand is a constant within the instruction
       addi $s1, $s2, 100

     – Register addressing: the operand is a register
       add $s1, $s2, $s3

     – Base or displacement addressing: the operand is at the memory location
       whose address is the sum of a register and a constant in the instruction
       lw    $s1, 100 ($s2)

     – PC-relative addressing: the address is the sum of the current PC and a
       constant (multiplied by 4) in the instruction
       beq $s1, $s2, 100

     – Pseudodirect addressing: the jump address is a constant (26 bits) in the
       instruction (multiplied by 4) concatenated with the upper 4 bits of the PC
       j      2500
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   8 of 69
 ISA Summary (cont.)
  • MIPS Addressing Modes (cont.)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   9 of 69
 The Five Classic Components of a Computer
•   Five classic components:
     – Memory: where instructions and
       data are kept
                                                                     Computer
     – Input: writes data to memory                                  Devices

     – Output: reads data from memory

                                                                     Memory
     – Datapath: performs arithmetic
       operations
                                                                     Processor
     – Control: sends the signals that
       determine the operations of the
       datapath, memory, input, and
       output according to instructions
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   10 of 69
 The Five Classic Components of a Computer (cont.)

   •   Processor: manipulates instructions and data

   •   A processor consists of datapath and control

   •   This computer organization is independent of hardware technology;
       you can place every piece of every computer into one of these five
       categories

   •   Underlying hardware in any computer performs the same basic
       functions:
        – inputting data,
        – outputting data,
        – processing data, and
        – storing data

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   11 of 69
 The Two Classes of Computer Architectures
   •   Different computer architectures are classified into one of the
       following two types:

   •   von Neumann (Princeton) architecture
        – Describes computers based on the following three key concepts:
           • Data and instructions are stored in a single read-write
             memory
           • The contents of this memory are addressable by location,
             without regard to the type of data contained there
           • Execution occurs in a sequential fashion (unless explicitly
             modified) from one instruction to the next

   •   Harvard architecture
        – traditionally: describes computers with separate memories;
          one for data and the other for instructions
        – today: describes computers with a single main memory, but
          with separate caches for instructions and data
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   12 of 69
 Clocking Methodology
 •   There are two different types of logic elements:
      1. elements that operate on data values (combinational), e.g., ALU
      2. elements that contain state (sequential), e.g., memory and registers


                     cycle time
                                                   rising edge falling edge
 •   A clocking methodology is used in synchronous systems to define when
     signals can be read and when they can be written
 •   An edge-triggered clocking methodology will be used:
      – Values stored in a state element are updated only on a clock edge
      – State elements update their internal storage on clock edges
      – As only state elements can store a data value, any collection of
         combinational logic must have its inputs come from a set of state
         elements and its outputs written into a set of state elements
      – The inputs are values written in a previous clock cycle, while the
         outputs are values that can be used in the following clock cycle
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   13 of 69
 Clocking Methodology (cont.)




  •   All signals must propagate from state element 1, through the
      combinational logic, and to state element 2 in the time of one cycle
  •   The time necessary for the signals to reach state element 2 defines
      the length of the clock cycle
  •   The value stored in a state element is changed only when the write
      control signal is asserted and a clock edge occurs
  •   Clock signals are not shown
  •   For simplicity, write control signals will not be shown when a state
      element is written on every active clock edge
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   14 of 69
 Clocking Methodology (cont.)




   •    An edge-triggered methodology allows doing all of the following in
        the same clock cycle:
         – reading the contents of a register
         – sending the value read through some combinational logic
         – writing back to that register

   •    Feedback cannot occur within one clock cycle because of the edge-
        triggered update of the state element

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   15 of 69
 Generic Implementation
 •   Much of what needs to be done to implement the instructions is the
     same, independent of the exact class of instruction
 •   For every instruction, the first two implementation steps are identical:
      – use PC to fetch the instruction from the instruction memory
      – use the instruction fields to select one or two registers to read:
          • lw requires reading only one register
          • j does not require accessing any register
          • All other instructions require reading two registers
 •   After these two steps, the actions required to complete the instruction
     depend on the instruction class

 •   ISA simplicity and regularity:
      – simplifies the implementation by making the execution of many of
        the instruction classes similar
          • Within a class, actions are largely independent of exact opcode
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   16 of 69
 Generic Implementation (cont.)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   17 of 69
 Generic Implementation (cont.)
  •   There are some similarities across different instruction classes, e.g.,
       – All instruction classes, except j, use the ALU after reading the registers:
             • Memory-reference instructions use the ALU for an address calculation
             • Arithmetic-logical instructions use it for the operation execution
             • The branch instruction (beq) uses it for comparison

       – After using the ALU, the actions required to complete various instruction
         classes differ:
           • A memory-reference instruction accesses the data memory either to
             write data for a store or read data for a load
           • An arithmetic-logical instruction writes the data from the ALU back
             into a register
           • A load instruction writes the data from the memory back into a
             register
           • A branch instruction may need to change the next instruction address
             based on the comparison; otherwise the PC should be incremented
             by 4 to get the address of the next instruction

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   18 of 69
 Generic Implementation (cont.)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   19 of 69
 Datapath Elements (1)
 •   Datapath element: A functional unit used to operate on or hold data within
     a processor
 •   Instruction memory:
      – Stores and supplies the instructions of a program given an address
      – No read signal is needed as it is read every clock cycle
      – This memory is not written
 •   Program counter (PC):
      – Holds the address of the current instruction (to be fetched)
      – No write signal is needed as it is written every clock cycle
 •   PC adder:
      – Used to increment the PC to the address of the next instruction
      – A combinational circuit that can be built from the ALU by writing the
         control line so that the control always specifies an add operation




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   20 of 69
 Building a Datapath (1)
   •   Instruction fetch and PC increment:
        – To execute any instruction, we must start by fetching the
          instruction from the instruction memory
        – To prepare for executing the next instruction, we must also
          increment the PC so that it points at the next instruction, 4
          bytes later




           Thick lines (with no
           width indication) indicate
           buses of 32 bits
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   21 of 69
 Datapath Elements (2)
   •   Register file:
        – A collection of the processor 32 general-purpose registers in
          which any register can be read or written by specifying its
          number in the file
        – The register file contains the register state of the computer

         – No read control signal is needed
            • The register file always outputs the contents of whatever
              register numbers are on the read register inputs
         – Writes are controlled by the write control signal (RegWrite)
            • must be asserted for a write to occur at the clock edge

         – Since writes to the register file are edge triggered, the design
           can legally read and write the same register within a clock
           cycle:
            • The read will get the value written in an earlier clock cycle
            • The value written will be available to read in a subsequent
               clock cycle
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   22 of 69
 Datapath Elements (2) (cont.)
   •   Arithmetic and logic unit:
        – The ALU is used to operate on the values read from the
          registers
        – It takes 2 32-bit inputs and produces a 32-bit result, as well as a
          1-bit signal if the result is zero
        – It is controlled by a 4-bit control signal (ALU operation)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   23 of 69
 Building a Datapath (2)
   •    Executing R-format instructions: (add $t1, $t2, $t3):
          – Read 2 data words from the register file, for each we need to input the
            register number (5 bits) to the register file and there will be an output
            (32 bits) that will carry the value that has been read from the registers
          – Perform an ALU operation on the contents of the registers
          – Write one data word result into the register file, for which we need two
            inputs: one (5 bits) to specify the register number to be written and
            another to supply the data (32 bits) to be written into the register

                                                                           4    ALU operation
                             Read
                             register 1           Read
                             Read                 data 1                 ALU
       Instr uctio n         register 2                                          Zero
                                           Registe rs                           ALU
                             Write                                             result
                             register             Read
                             Write                data 2
                             data
                                           Reg Write

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   24 of 69
 Datapath Elements (3)
   •   Sign-extension unit:
        – Sign-extends the 16-bit offset field in the instruction to a 32-bit
          signed value
        – Replicates the high-order sign bit of the original data item in the
          high-order bits of the larger, destination data item
   •   Data memory:
        – To be read from or written to
        – It must be written on store instructions; hence, it has both read
          and write control signals, an address input, and an input for the
          data to be written into memory




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   25 of 69
 Building a Datapath (3)
  •       Executing loads/stores (lw $t1, 4($t2)/sw $t1, 4($t2)):
             – Compute a memory address by adding the base register to the
               16-bit signed offset field contained in the instruction
             – If the instruction is a store, the value to be stored must also be
               read from the register file
             – If the instruction is a load, the value read from memory must be
               written into the register file in the specified register
                                                                  4    A L U o p e r a ti o n
                       Read
                       re gi s te r 1                                                                              M e m W ri t e
                                                 Read
                                                 d a ta 1
                       Read
 I n s tr u c ti o n   re gi s te r 2                                    Zero
                                        R e giste r s             ALU A L U
                       W ri t e                                      r e s ul t                    A d d re s s           Read
                       re gi s te r                                                                                       da ta
                                                 Read
                       W ri t e                  d a ta 2
                       d a ta                                                                                      D a ta
                                                                                                                  m e m o ry
                       R e g W ri t e                                                              W ri t e
                                                                                                   d a ta
                                        16                   32
                                                  S ig n                                                           Me m Read
                                                e x te n d




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.                              Chapter 4.Part I               26 of 69
 Building a Datapath (3) (cont.)
  •   Executing R-format instructions and memory instructions:




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   27 of 69
 Building a Datapath (4)
   •   Executing beq instructions (beq $t1, $t2, 5):
        – Compute the branch target address by adding the sign-extended
          offset field of the instruction to the PC that was incremented by 4
            • Shift left the sign-extended offset by 2 bits as it is a word offset
        – Compare the register contents to determine whether the next
          instruction is the one that follows sequentially or the instruction at the
          branch target address
            • Send the two register operands to the ALU to subtract them
            • If the two operands are equal, the Zero signal out of the ALU will
              be asserted
            • If the condition is true (operands are equal), the branch target
              address becomes the new PC (the branch is taken)
            • If the condition is false (operands are not equal), the incremented
              PC replaces the current PC (the branch is not taken)
   •   Executing j instructions (j 95):
        – Replace the lower 28 bits of the PC with the lower 26 bits of the
          instruction shifted left by 2 bits (concatenate 00 to the jump offset)
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   28 of 69
 Building a Datapath (4) (cont.)




   • J instructions are not supported in this diagram
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   29 of 69
 Creating a Single Datapath
   •   The simplest datapath executes all instructions in one clock cycle

   •   This means that no datapath resource can be used more than once
       per instruction
        – Any element needed more than once, must be duplicated

   •   We therefore need a memory for instructions separate from the one
       for data

   •   Many of the datapath elements can be shared by different
       instruction flows

   •   A multiplexor (data selector) is used to share a datapath element
       between two different instruction classes
        – It allows multiple connections to the input of an element
        – It has a control signal to select among the multiple inputs
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   30 of 69
 Creating a Single Datapath (cont.)
   •   The key difference between the three datapaths are:
        – The second input to the ALU unit is one of two things, it is either
            • a register, if it is an R-format instruction or
            • the sign-extended lower half of the instruction, if it is a lw or sw
              instruction
            • So, place a multiplexor at the second input to the ALU unit
        – The value stored into a destination register comes from
            • the ALU, if it is an R-format instruction or
            • memory, if it is a memory load instruction
            • So, place a multiplexor at the data input to the register file

   •   A multiplexor (with a selector control signal RegDst) is needed to select
       the field of instruction used to indicate the register number to be written
        – bit positions 20:16 (rt) for lw or
        – bit positions 15:11 (rd) for an R-format instruction

   •   Add the instruction fetch portion of the datapath
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   31 of 69
 Creating a Single Datapath (cont.)
  •   The branch instruction uses the main ALU for comparison of the
      register operands
       – So, an adder is needed for computing the branch target address
       – An additional multiplexor is required to select either the
          sequentially following instruction address (PC+4) or the branch
          target address to be written into the PC

  •   PCSrc is set if the instruction is beq and the ALU Zero output is true
       – PCSrc is generated by ANDing together the Branch signal from
         the control unit and the Zero signal out of the ALU




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   32 of 69
 Creating a Single Datapath (cont.)




   • J instructions are not supported in this diagram
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   33 of 69
 Creating a Single Datapath (cont.)


                                                                                  PCSrc




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I     34 of 69
 The ALU Control
 •   ALU has four control inputs

                          ALU control lines             Function
                               0000                       and
                               0001                        or
                               0010                       add
                               0110                       sub
                               0111                       slt
                               1100                       nor

 •   Depending on the instruction class, the ALU will need to perform one
     of the first five functions:
      – lw and sw: the ALU computes the memory address by addition
      – R-format: the ALU performs one of the actions (and, or, sub,
         add, slt), depending on the value of the 6-bit funct field
      – beq: the ALU must perform a subtraction
 •   nor is needed for other parts of the MIPS ISA
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   35 of 69
 The ALU Control (cont.)
   •   A small ALU control unit (combinational circuit) generates the 4-bit ALU
       control input
        – This unit has as inputs the instruction funct field of the instruction and
           a 2-bit control field (ALUOp) from the main control unit
        – ALUOp indicates whether the operation to be performed should be:

            add (00)                                   for loads and stores
            sub (01)                                   for beq
            determined by funct (10)                   for R-format

   •   Multiple levels of decoding style: e.g., the main control unit generates
       the ALUOp bits, which then are used as input to the ALU control that
       generates the actual signals to control the ALU unit:
        – can reduce the size of the main control unit
        – may potentially increase the speed of the control unit by using several
           smaller control units
        – These optimizations are important, since the control unit is often
           performance-critical (clock cycle time)
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   36 of 69
 The Main Control Unit
  •   The control unit takes inputs and generates a write signal for state
      elements, the selector control for multiplexors, and the ALU control

  •   The control unit uses the 6-bit opcode field, Op [5:0], to generate
      these control signals

  •   MIPS ISA regularity and simplicity means that a simple decoding
      process can be used to determine how to set the control lines

  •   Control unit implementation is combinational

  •   The datapath operates in a single clock cycle and the signals within
      the datapath can vary unpredictably during the clock cycle

  •   Signals stabilize in the order of the flow of information
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   37 of 69
 The Main Control Unit (cont.)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   38 of 69
 Individual Exercise (1)
   •    Determine whether any of the control signals in the
        single-cycle implementation can be eliminated and
        replaced by another existing control signal, or its inverse




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   39 of 69
 Individual Exercise (1): Answer
   •    Determine whether any of the control signals in the
        single-cycle implementation can be eliminated and
        replaced by another existing control signal, or its inverse




        – RegDst can be replaced by ALUSrc’, MemtoReg’, MemRead’, ALUop1

        – MemtoReg can be replaced by RegDst’, ALUSrc, MemRead, ALUOp1’

        – Branch and ALUOp0 can replace each other

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   40 of 69
 Individual Exercise (2)
   •    A friend is proposing to modify the single-cycle datapath
        by eliminating the control signal MemtoReg
   •    The multiplexor that has MemtoReg as an input will
        instead use either the ALUSrc or the MemRead control
        signal

   •    Will your friend's modification work?
   •    Can one of the two signals (ALUSrc and MemRead)
        substitute for the other? Explain




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   41 of 69
 Individual Exercise (2): Answer
   •    A friend is proposing to modify the single-cycle datapath by
        eliminating the control signal MemtoReg. The multiplexor that has
        MemtoReg as an input will instead use either the ALUSrc or the
        MemRead control signal. Will your friend's modification work?
       –    MemtoReg looks identical to both ALUSrc and MemRead signals,
            except for the don’t care entries, which have different settings
            for the other signals
       –    A don’t care can be replaced by any signal; hence both signals
            can substitute for the MemtoReg signal


   •    Can one of the two signals (ALUSrc and MemRead) substitute for
        the other? Explain
       –    Signals ALUSrc and MemRead differ in that sw sets ALUSrc (for
            address calculation) and resets MemRead (writes memory:
            cannot have a read and a write in the same cycle), so they
            cannot replace each other
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   42 of 69
 The Operation of the Datapath
   •    Although everything occurs in one clock cycle, we can think of four
        steps to execute the instructions
   •    These steps are ordered by the flow of information

   •    Execution steps for R-format instructions:
        1. The instruction is fetched from the instruction memory, and the
           PC is incremented
        2. Two register values are read from the register file using bits
           25:21 and 20:16 of the instruction to select the source
           registers
            a) also, the main control unit computes the setting of the
                control lines during this step
        3. The ALU operates on the data values read from the register
           file, using the function code (bits 5:0, which is the funct field,
           of the instruction) to generate the ALU function
        4. The result from the ALU is written into the register file using
           bits 15:11 of the instruction to select the destination register
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   43 of 69
 The Operation of the Datapath (cont.)
   •    Execution steps for R-format instructions (cont.)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   44 of 69
 The Operation of the Datapath (cont.)
   •    Execution steps for a lw instruction:
        1. The instruction is fetched from the instruction memory, and the
           PC is incremented
        2. A register value is read from the register file using bits 25:21 of
           the instruction to select the source register
            a) also, the main control unit computes the setting of the
                control lines during this step
        3. The ALU computes the sum of the value read from the register
           file and the sign-extended, lower 16 bits of the instruction
        4. The sum from the ALU is used as the address for the data
           memory
        5. The data from the memory unit is written into the register file
           using bits 20:16 of the instruction to select the destination
           register


Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   45 of 69
 The Operation of the Datapath (cont.)
   •     Execution steps for a lw instruction (cont.):




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   46 of 69
 The Operation of the Datapath (cont.)
   •    Execution steps for a beq instruction:
        1. The instruction is fetched from the instruction memory, and the
           PC is incremented
        2. Two register values are read from the register file using bits
           25:21 and 20:16 of the instruction to select the source registers
            a) also, the main control unit computes the setting of the
               control lines during this step
        3. The ALU performs a subtract on the data values read from the
           register file
            a) The value of PC+4 is added to the sign-extended, lower 16
               bits of the instruction shifted left by two, the result is the
               branch target address
        4. The Zero result from the ALU is used to decide which adder
           result to store into the PC


Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   47 of 69
 The Operation of the Datapath (cont.)
   •     Execution steps for a beq instruction (cont.):




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   48 of 69
 Home Exercise (3)
   •    Show how each of the following instructions uses the datapath

        add         $t1, $t2, $t3
        lw          $t1, 04 ($t2)
        beq         $t1, $t2, 34

   •    Work out the R-format example on Page 266 and in Figure 4.19
        step by step in the textbook!

   •    Work out the lw example on Page 267 and in Figure 4.20 step by
        step in the textbook!

   •    Work out the beq example on Page 268 and in Figure 4.21 step by
        step in the textbook!



Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   49 of 69
 Home Exercise (4)
   •    In the single-cycle datapath, each instruction uses a
        datapath element to carry out its execution

   •    Many of the datapath elements operate in series, using
        the output of another element as an input

   •    Some datapath elements operate in parallel

   •    Show which elements operate in parallel




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   50 of 69
 Implementing Unconditional Jumps
   •   An additional multiplexor is used with a new control signal: jump


                                                                          2
                                                                                               5
                1                                                                       4




                                                            2                                 3



                                   1




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I       51 of 69
 Group Exercise (5)
   •    We wish to add the instruction addi to the single-cycle
        datapath
   •    Add any necessary datapaths and control signals to the
        single-cycle datapath in Figure 4.24 (Slide 51)
   •    Show the necessary additions to the table in Figure 4.18
        (Slide 38)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   52 of 69
 Group Exercise (5): Answer
  •       We wish to add the instruction addi to the single-cycle datapath
  •       Add any necessary datapaths and control signals to the single-cycle
          datapath in Figure 4.24 (Slide 51)
  •       Show the necessary additions to the table in Figure 4.18 (Slide 38)

  •      No additions to the datapath are required
  •      The new control is similar to
         – lw as we want to use the ALU to add the immediate to a register
         – an R-format instruction as we want to write the result of the
            ALU into a register and we are not branching or using memory

                                Memto-   Reg-    Mem-   Mem-
Instruction   RegDst   ALUSrc                                   Branch    ALUOp1     ALUOp0    Jump
                                 Reg     Write   Read   Write
 R-format       1        0        0       1       0      0        0          1          0        0
    lw          0        1        1       1       1      0        0          0          0        0
   addi         0        1        0       1       0      0        0          0          0        0

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.       Chapter 4.Part I     53 of 69
 Group Exercise (6)
   •    We wish to add the instruction sll to the single-cycle
        datapath
   •    Add any necessary datapaths and control signals to the
        single-cycle datapath in Figure 4.24 (Slide 51)
   •    Show the necessary additions to the table in Figure 4.18
        (Slide 38)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   54 of 69
 Group Exercise (6): Answer
  •     We wish to add the instruction sll to the single-cycle datapath
  •     Add any necessary datapaths and control signals to the single-cycle
        datapath in Figure 4.24 (Slide 51)
  •     Show the necessary additions to the table in Figure 4.18 (Slide 38)

  •     We need to add a new operation to the ALU
  •     We need to input the shamt field (Instruction [10:6]) to the ALU

 ALUOp        Funct           ALU control                            Control action
                                                  Shift the second ALU operand ($rt) by
             00 0000              1110
   10                                             the amount in the shamt field
               (sll)       (shift left logical)
                                                  (Instruction [10-6]), input to the ALU




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.      Chapter 4.Part I   55 of 69
 Group Exercise (6): Answer (cont.)




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   56 of 69
 Inefficiencies in the Single-Cycle Implementation
  •   The clock cycle must have the same length for every instruction in the
      single-cycle design and the CPI will therefore be 1
  •   The clock cycle is determined by the critical path:
       – Critical path: the longest possible path in the processor
       – This path is almost certainly a load instruction, which uses 5 functional
          units in series:
            1. the instruction memory,
            2. the register file (read),
            3. the ALU,
            4. the data memory, and
            5. the register file (write)

  •   The clock cycle is assumed equal to the worst-case delay for all instructions
       – It is useless to try implementation techniques that reduce the delay of
          the common case but do not improve the worst-case cycle time
       – A single-cycle implementation thus violates the key principle of making
          the common case fast
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   57 of 69
 Inefficiencies in the Single-Cycle Implementation (cont.)

   •   Although the CPI is 1, the overall performance of a single-cycle
       implementation is likely to be poor, since the clock cycle is too long
       and several of the instruction classes could fit in a shorter clock
       cycle

   •   If we had a processor with more powerful and complicated
       operations (like floating-point instructions) and addressing modes,
       instruction delays could vary a lot

   •   With this single-cycle implementation, each functional unit can be
       used only once per clock; therefore, some functional units must be
       duplicated, raising the cost of the implementation and wasting area

   •   So, a single-cycle implementation is inefficient in both its
       performance and its hardware cost!

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   58 of 69
 Inefficiencies in the Single-Cycle Implementation (cont.)

   •   The penalty for using the single-cycle design with a fixed clock cycle
       is significant

   •   Solutions:
        – Using variable length clocks, which is extremely difficult, and
          the overhead of it could be larger than any advantage gained!
        – Using implementation techniques that have a shorter
          clock cycle and that require multiple clock cycles for each
          instruction. These techniques do less work every cycle and then
          vary the number of clock cycles for different instruction classes
        – Use pipelining that uses a datapath very similar to the single-
          cycle datapath, but is much more efficient
              • It gains efficiency by overlapping the execution of multiple
                instructions, increasing hardware utilization and throughput
              • It improves performance
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   59 of 69
 Group Exercise (7)
  •    Assume that the operation times for the major functional
       units in the single-cycle implementation are the
       following:
       – 200 ps for memory access,
       – 200 ps for ALU and adders operation,
       – 100 ps for register file read or write,
       – multiplexors, control unit, PC accesses, sign extension
           unit, and wires have no delay

  •    What is the minimum clock period for an implementation
       in which every instruction operates in 1 clock cycle of a
       fixed length?

Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   60 of 69
 Group Exercise (7): Answer
   •    The critical path for the different instruction classes is as follows:
  Instruction class                  Functional units used by the instruction class
 Load word (lw)        Instruction fetch    Register access   ALU   Memory access   Register access
 Store word (sw)       Instruction fetch    Register access   ALU   Memory access
 R-format (ALU)        Instruction fetch    Register access   ALU                   Register access
 Branch (beq)          Instruction fetch    Register access   ALU
 Jump (j)              Instruction fetch

   •    Using these critical paths, the required clock cycle for each instruction is:
       Instruction       Instruction       Register      ALU          Data    Register     Total
          class             fetch           read       operation     access    write       time
Load word (lw)              200 ps          100 ps       200 ps      200 ps    100 ps      800 ps
Store word (sw)             200 ps          100 ps       200 ps      200 ps                700 ps
R-format (ALU)              200 ps          100 ps       200 ps                100 ps      600 ps
Branch (beq)                200 ps          100 ps       200 ps                            500 ps
Jump (j)                    200 ps                                                         200 ps

   •    The clock cycle for a processor with a single clock for all instructions will be
        determined by the longest instruction, which is 800 ps
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.     Chapter 4.Part I    61 of 69
 Group Exercise (8)
  •    Assume the following instruction mix for the processor in Exercise
       (7): 25% loads, 10% stores, 45% ALU instructions, 15% branches,
       and 5% jumps


  •    Which of the following implementations would be faster and by how
       much?
       1. An implementation in which every instruction operates in 1 clock
          cycle of a fixed length
       2. An implementation, where every instruction executes in 1 clock
          cycle using a variable-length clock cycle, which for each
          instruction is only as long as it needs to be


  •    Show which units can tolerate more delays in the single-cycle
       implementation. Quantify your answer.


Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   62 of 69
 Group Exercise (8): Answer (cont.)
   •   The clock cycle for a processor with a single clock for all instructions will be
       determined by the longest instruction, which is 800 ps

   •   A processor with a variable clock will have a clock cycle that varies between
       200 ps and 800 ps, which could be calculated as follows:

       CPU clock cycle = 800*0.25+700*0.10+600*0.45+500*0.15+200*0.05
                       = 625 ps

   •   CPI is 1 for the two implementations
   •   The instruction count is the same for the two implementations

   •   So what matters only is the clock cycle time

       Performanc e                        Execution time   single clock       CPIxCx 800
                      variable clock
                                       =                                   =              = 1.28
       Performanc e    sin gle clock
                                           Execution time
                                                        variable clock
                                                                               CPIxCx 625



Patterson and Hennessy’s Computer Organization and Design, 5th Ed.                           Chapter 4.Part I   63 of 69
 Group Exercise (8): Answer (cont.)
   •   Units that are not on the critical path can tolerate more delays

   •   Load instructions are on the critical path that includes the following
       functional units: instruction memory, register file read, ALU, data
       memory, and register file write
   •   Increasing the delay of any of these units will increase the clock
       period of this datapath

   •   The units that are outside this critical path (with non-zero delays)
       are the two adders used for PC calculation: (PC + 4) and (PC +
       immediate field), which produce the branch outcome
   •   The delay of the two adders (equivalent to two ALUs) is 400 ps

   •   Thus, the sum of the two adder’s delay can tolerate delays up to
       the clock period – 400 ps = 800 ps – 400 ps = 400 ps
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   64 of 69
 Group Exercise (9)
   •    Suppose the processor in Exercise (7) has a floating-point ALU that requires
        – 400 ps for a floating-point add or subtract and
        – 600 ps for a floating-point multiply or divide
   •    Any floating-point instruction uses this floating-point ALU
   •    Assume the following:
        – loads comprise 30% of the instruction mix,
        – stores comprise 15% of the instruction mix,
        – integer R-format instructions comprise 25% of instruction the mix,
        – branches comprise 10% of the instruction mix,
        – jumps comprise 5% of the instruction mix, and
        – together floating-point add and subtract comprise 5% of the mix, while
        – together floating-point multiple and divide comprise 10% of the mix
        – All instructions of the same class take the same time to execute

   •    Find the performance ratio between an implementation in which the clock
        cycle is different for each instruction class and another one in which all
        instructions have the same clock cycle time
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   65 of 69
 Group Exercise (9): Answer
      Instruction            Instruction          Register              ALU                Data      Register    Total
         class                  fetch              read               operation           access      write      time
Load word (lw)                        200 ps       100 ps                 200 ps          200 ps      100 ps     800 ps
Store word (sw)                       200 ps       100 ps                 200 ps          200 ps                 700 ps
R-format (ALU)                        200 ps       100 ps                 200 ps                      100 ps     600 ps
Branch (beq)                          200 ps       100 ps                 200 ps                                 500 ps
Jump (j)                              200 ps                                                                     200 ps
FP add/subtract                       200 ps       100 ps                 400 ps                      100 ps     800 ps
FP multiply/divide                    200 ps       100 ps                 600 ps                      100 ps     1000 ps
  •   The clock cycle for a processor with a single clock for all instructions will be
      determined by the longest instruction, which is floating-point multiply or
      divide, which is 1000 ps
  •   A processor with a variable clock will have a clock cycle that varies between
      200 ps and 1000 ps, which could be calculated as follows:
      CPU clock cycle = 800*0.30+700*0.15+600*0.25+500*0.10+200*0.05
                       + 800*0.05+1000*0.10 = 695 ps
      Performanc e                        Execution time   single clock       CPIxCx1000
                     variable clock
                                      =                                   =              = 1.44
      Performanc e    single clock
                                          Execution time
                                                       variable clock
                                                                              CPIxCx 695
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.                            Chapter 4.Part I    66 of 69
 Group Exercise (10)
   •    Assume the operation times for the major functional units used in Exercise
        (7) and instruction mix in Exercise (8)

   •    Instead of a single-cycle organization, we can use a multicycle
        organization, where each instruction takes multiple cycles but one
        instruction finishes before another is fetched

   •    In this organization, an instruction only goes through units it actually
        needs

   •    Compare the clock cycle time and execution time of this organization with
        the single-cycle organization in Exercise (7)

   •    Assume that the timing overhead in this organization is ignored




Patterson and Hennessy’s Computer Organization and Design, 5th Ed.   Chapter 4.Part I   67 of 69
 Group Exercise (10): Answer (cont.)
 Instruction
                                       Actions done each cycle by the instruction class
    class
 Clock cycle            1                   2                   3                   4                 5
 Load word                                                     ALU            Memory access     Register access
 Store word                                                    ALU            Memory access
 R-format        Instruction fetch                             ALU            Register access
                                      Register access
                Address calculation                            ALU
 Branch                                                                         PC change
                                                        Address calculation
 Jump                                                       PC change
   Delay               200                 100                 200                 200               100

  •     In the first two cycles, we do not yet know what the instruction is, so we can
        perform only actions that are applicable to all instructions
  •     Address calculation in the third cycle calculates the branch-to address to be
        used if the instruction is a beq
  •     PC change in the third and fourth cycles has no delay
  •     Tasks done in the same cycle are done in parallel
        –      The slowest task in a cycle determines the clock period needed for that cycle
  •     Clock cycle time is 200 ps
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.                Chapter 4.Part I        68 of 69
 Group Exercise (10): Answer (cont.)
        Instruction class            Cfraction     Number of clock cycles (CPI)
       Load word (lw)                  25%                              5
       Store word (sw)                 10%                              4
       R-format (ALU)                  45%                              4
       Branch (beq)                    15%                              4
       Jump (j)                         5%                              3
         n
CPI =  (CPI i x C fractioni ) = (5 x 0.25) + (4 x 0.10) + (4 x 0.45) + (4 x 0.15) + (3 x 0.05) = 4.20
        i =1

CPU time = CPI × C × T = 4.20 × C × 200 = 840 x C ps

   •     Note: in this example, it just happened that the operation time for
         the major functional units and instruction mix used make
         multicycle organization slower than the single-cycle one. This is
         not always the case!
Patterson and Hennessy’s Computer Organization and Design, 5th Ed.       Chapter 4.Part I     69 of 69

