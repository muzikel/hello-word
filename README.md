= Verilog coding guidelines =
Updated 16:36, 22 January 2007 (PST)

This is a GUIDE for writing Verilog for synthesis. As such it is a list of
suggestions and recommendations, not rules. Some suggestions are very
subjective while others are almost mandatory - i.e. you should have a good
reason for not following them. 

== Naming ==

=== Module/instance naming ===

You are free to name your modules as you please, provided that the following conditions are met:
* The final top level modules should be the chip name with  <tt>_top</tt>  appended, e.g.  <tt>unet_top.v</tt> .
* When instantiating a module the instance name should be either:
** identical to the module name if the module name adequately describes the purpose. e.g.  <tt>sram_output_fifo sram_output_fifo (...);</tt> 
** the module name followed by a descriptive subscript. e.g.  <tt>fifo fifo_sram_output (...);</tt> 

=== Signal naming ===

* All signals are lowercase alpha, numeric and underscore only. (Exceptions permitted for external pins and auto-generated code.)
* Avoid active low signals in the core.  If you must have them then append  <tt>_n</tt>  to the signal name. e.g.  <tt>active_low_b</tt> 
<blockquote>
 (Active low signals at the pins are fine but invert them in the core before using them.) <br />
</blockquote>
* Buses are numbered high order to low order:  <tt>bus[31:0]</tt> 
* Array locations are numbered low to high. e.g. 2048x32 bit RAM:
<pre> reg [31:0] mem_array [0:2047];
</pre>
*  '''Don't''' try to indicate the direction of the signal in the name: e.g.  <tt>my_signal_in</tt>  or  <tt>my_signal_i</tt>  is  '''bad''' . Just use  <tt>my_signal</tt> . This makes it much easier to hook up blocks.
*  '''Do''' use the same name for the signal at all levels of hierarchy.
*  '''Do''' indicate the direction of a signal as a comment in the signal list:
<pre> my_module (
     my_signal,  //I: Input from other_module
 );
</pre>

=== Constant naming ===

* Use capitals to identify constants: e.g.  <tt>DEBUG</tt> ,  <tt>IDLE</tt> 

== Indenting ==

* Use three spaces per indentation level. Expand tabs to spaces to prevent indentation changing based upon editor settings.
<blockquote>
 Vim users can place the following modeline near the  ''top'' or  ''bottom'' of each file: <br />
</blockquote>
<pre> // vim:set shiftwidth=3 softtabstop=3 expandtab:
</pre>
<blockquote>
 Emacs users <br />
</blockquote>
* Indent begin/end statements as follows:
<pre> if (this) begin
     do something;
 end
 else begin
     do something else;
 end
</pre>

== Commenting ==

* Comment your code. Someone will be coming back to this code in 6 months time, and that person will probably be  '''you''' .
* When entering multi-line comments, try to use the "/*..*/" commenting rather than putting "//" in front of every line.  It makes editing much easier.
* Add the following header to the top of all source code written by us (change the details as necessary):
<pre> ///////////////////////////////////////////////////////////////////////////////
 // vim:set shiftwidth=3 softtabstop=3 expandtab:
 // $Id: reg_file.v 916 2006-04-20 23:23:25Z bob $
 //
 // Module: reg_file.v
 // Project: CPCI (PCI Control FPGA)
 // Description: Register file for access via PCI
 //
 //
 // Change history: 8/18/04 - Implemented DWORD0 (Revision/Version ID)
 //                 10/29/04 - Made the RESET signal last longer than
 //                 a single clock
 //                 01/08/05 - dma_rd_mac is now an input as round-robin
 //                 lookup is used
 //                 01/13/05 - split dma_intr into read and write sigs
 //
 ///////////////////////////////////////////////////////////////////////////////
</pre>
''Note:'' We probably need to add copyright info to the header as well.

== Module declaration/instantiation ==

* Each module should be defined in a separate file.
* Use Verilog 2001 ANSI-C style port declarations:
<pre> my_module (
    input signal_A,
    input signal_B,
 
    output reg signal_C,
 
    input clk
 );
</pre>
* Declare inputs and outputs one per line. It makes  <tt>grep</tt> 'ing so much easier. If I want to find who drives a signal in code written this way, I can grep for " <tt>output.*signal</tt> ". It also gives you a handy place to put a comment.
* Group signals logically by their function (e.g. keep DDR signals together). Place the 'miscellaneous' signals at the end of the signal list. This will include  <tt>clk</tt> ,  <tt>reset</tt>  and anything else that generally has a high fanout and goes between many modules.
<pre> my_module (
     signals_to_from_block_A, // description
 
     signals_to_from_block_B, // description
 
     input reset,
     input clk
 );
</pre>
* Instantiate signals one per line whether as inputs or when instantiating a block&mdash;it makes it easier on the scripts!
*  '''Don't''' instantiate modules using positional arguments.  '''Always''' use the dot form:
<pre> my_module my_module(
     .signal (signal),
     .a_bus  (a_bus),
     ...
 );
</pre>
<blockquote>
 (There are some utilities for building instantiations of modules that work if the signal names are consistent between levels.) <br />
</blockquote>
* Use  '''explicit''' in-line parameter redefinition when overriding parameters:
<pre> my_module my_module  '''#(.WIDTH(32))''' (
     ...
 );
</pre>

== Clocking ==

* Core clock signal for a module is ' <tt>clk</tt> '. 
* All other clocks should have a description of the clock and the frequency embedded&mdash;it makes life easier for the synthesis person. e.g.  <tt>clk_ddr_400</tt> 
* Keep to one clock per module if possible. (It's easier to understand and it will synthesize much faster.)

=== Synchronization/clock domain crossing ===

* If you have signals that need to be synchronized (cross asynchronous clock boundaries) then name the synch flops as  <tt>synch_&lt;something&gt;</tt> . e.g.
<pre> reg synch_stage_1, synch_stage_2;
</pre>
<blockquote>
 These can be given special treatment for simulation, and you can apply automatic checks to ensure all synch domains have synch flops. <br />
 Corollary:  '''Don't''' use ' <tt>synch</tt> ' in any other flop names. <br />
</blockquote>
* Avoid mixing synchronization/domain crossing code with general logic. Clearly delineate the code in a large module with other logic or place in a separate module. 

* The statements " <tt>input</tt> " and " <tt>output</tt> " will default their arguments to wires, so there is no need to put a wire statement in the code for anything that is an input or an output.  Other wires (hookup wires) need to be declared only if they are multi-bit, although some people like to declare the single-bit ones as well.

== Reset ==

Reset signal is:
* called ' <tt>reset</tt> ' 
* active high within the core
* synchronous

== Assignment statments ==

* Sequential elements  '''MUST''' only have non-blocking assignments (<code><=</code>)
* Combinatorial should only have blocking assignments (<code>=</code>)
* Don't mix blocking and non-blocking assignments within a code block.

== Parameters, defines and constants ==

* Do parameterize modules if appropriate and if readability is not degraded.
* Propagate parameters through the hierarchy:
<pre> #(parameter ADDR_WIDTH = 10,
   parameter DATA_WIDTH = 32)
 ( input [DATA_WIDTH-1:0] data,
   input [ADDR_WIDTH-1:0] addr,
   ...
 );
 ...
 /* instance using the same parameters */
 b_module b_module_0
 #(.ADDR_WIDTH(ADDR_WIDTH),
   .DATA_WIDTH(DATA_WIDTH))
 ( .data(data)...);
</pre>
* Place all global  <tt>`define</tt> s in external defines files that are included in the project.
* Do not declare  <tt>`define</tt>  statements in individual modules.
* Use  <tt>localparam</tt>  for constants that should not be redefined from outside of the module. E.g. the states for a state machine:
<pre> localparam IDLE    = 3'd0; // Oven idle
 localparam RAMP_UP = 3'd1; // Oven temperature ramping up
 localparam HOLD    = 3'd2; // Hold oven at bake temperature
 ...
</pre>

== Debugging & assertions ==

* Group your assertions at the end of the module, surrounded by  <tt>synthesis translate_off</tt>  and  <tt>synthesis translate_on</tt> :
<pre> ... // User code here
 
 // synthesis translate_off
 always @(posedge clk)
    // Verify that the length never exceeds 10
    if (len > 10)
       $display($time, " ERROR: Length exceeds 10 in %m");
 // synthesis translate_on
 
 endmodule
</pre>

== State machines ==

* Simple state machines: feel free to combine flops with next-state combinational logic
* Long/complex state machines: separate flops from combinational/next state logic. e.g.
<pre> always @(posedge clk)
 begin
     state <= state_nxt;
     addr <= addr_nxt;
 end
 
 always @*
 begin
     // Default to current state
     state_nxt = state;
     addr_nxt = addr;
 
     if (reset)
     begin
         state_nxt = START_STATE;
         addr_nxt = 'h0;
     end
     else
     begin
         case (state)
             START_STATE: begin
                 if (ready)
                     state_nxt = GO_STATE;
             end
             ...
         endcase
     end
 end
</pre>
* Assign default values at the start of conditional statements: this tends to shorten the code (more readable) and helps to ensure that everything always gets a value.
<pre> always @* begin
    a = default_a;
    b = default_b;
 
    case (expr)
       state_1 : ... // if a should be default_a then we dont put it here.
       state_2 : ...
    endcase
</pre>

== General Verilog coding ==

* Synthesizable logic goes in  ''leaf'' modules.
** As much as possible, try to keep all synthesizable code in leaf modules, and use hierarchical modules  '''only''' to hook things up.  A single "!" at core can wreak havoc.  
* Use @* for the sensitivity list for combinational logic always blocks: 
<pre> always @*
    a = b + c;
</pre>
* Don't try to write ultra-slick Verilog&mdash;it makes it harder to read.
* Avoid for loops&mdash;they are usually difficult to understand.  ''Sometimes'' they are really useful: e.g. unrolling bit expressions that involve other bits.
*  = '''generate''' =  blocks are encouraged for generating instances/blocks of code:
<pre> generate
    genvar i;
    for (i = 0; i < 4; i = i + 1) begin: par_gen
       parity8 parity(
          .data(wr_data[(i * 8) +: 8]),
          .parity(wr_par[i])
       );
    end
 endgenerate
</pre>
* Don't use the  <tt>`include</tt>  directive in synthesizable code. This is never necessary and introduces synthesis dependencies that are difficult to track.
** If an include absolutely cannot be avoided then surround included file with  <tt>`ifdef</tt> ,  <tt>`define</tt>  and  <tt>`endif</tt>  directives.
* No  '''tri-states''' in the core!  Only tri-state I/Os! 
* Use small-ish modules&mdash;try to keep modules small and readable.

== Synthesis ==

*  '''NO LATCHES!''' Check the results of synthesis and ensure you do not have any inferred latches&mdash;it usually means you missed something in your coding.
*  '''No''' asynchronous logic/combinatorial feedback loops/self timed logic.
* Try to flop all inputs and outputs to a block&mdash;it makes it much easier to set timing constraints. This means you will have to think about how modules are grouped for synthesis.  It's fine not to have flops in a module if that module is going to be synthesized with the modules that do have the relevant source/sink flops.
* All non data-path flops must have a reset.
* All data-path flops must reset within 5 clocks. Usually this means that you only need to reset the input pipeline flops, and subsequent flops will reset within the specified number of clocks.  If you're not sure then reset them explicitly.
* If you do have some flops with reset and some without, then don't put them both in the same block of code:
<blockquote>
 BAD: <br />
</blockquote>
<pre> always @(posedge clk) 
     if (reset) 
         stage_1 <= 'h0;
     else begin
         stage_1 <= input;
         stage_2 <= stage_1;
     end
</pre>
<blockquote>
 This causes reset to be used as an enable on stage_2. Move stage_2 to a separate block or take it out of the if-then-else. <br />
 GOOD: <br />
</blockquote>
<pre> always @(posedge clk) 
    if (reset) 
       stage_1 <= 'h0;
    else
       stage_1 <= input;
 
 always @(posedge clk)
    stage_2 <= stage_1;
</pre>
* Specify  '''ALL''' cases in a case statement (use default if you don't need them all).
<blockquote>
 Bear in mind that XST will prioritize in top-to-bottom order. <br />
</blockquote>
*  '''Don't''' use  <tt>casex</tt>  and  <tt>casey</tt> 
* Do  '''not'' use pragmas for case:  <tt>parallel_case</tt>   <tt>full_case</tt> 
* Keep it simple. If you cannot envisage the final synthesized logic for your code then XST will probably have a hard time.
* Don't mix hand-instantiated circuitry with RTL

== Acknowledgements, further reading, and more ==

These guidelines are based on a document written by Greg Watson. 

Many of these recommendations are courtesy of Paul Zimmer.

For further reading on Verilog styles, synthesis techniques, useful
scripts, etc I recommend the following papers:

* "RTL Implementation Guide", Jack Marshall, Tera Systems: http://www.terasystems.com/products/Marshall_RTL_Guide.pdf
* Cliff Cummings' papers at: http://www.sunburst-design.com/papers/
* Steve Golson's papers at: http://www.trilobyte.com/papers/

Many of the above papers appeared at SNUG:
http://www.snug-universal.org/papers/papers.htm
