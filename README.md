# risc5-single-core-cycle
system verilog code for risc5 core cycle
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////

//////////////////////////////////////////////////////////////////////////////////


module top_core(
//Programm_counter
input logic           clk,
input logic           reset
);
logic           branch_i;
logic  [1:0]    nextpc_sel;
logic [31:0]    imm_i;
logic [31:0]    pc_out;
logic [31:0]    program_counter;

//Inst_mem
logic       read_en_i;
logic  [11:0]   addr_i; 
logic [31:0]   data_i;
logic [31:0]   data_o;

//Control_unit
logic  [31:0] ins;
logic  [31:0] pc;
logic         illegal;
logic         memtoreg; 
logic         regwrite; 
logic         memread; 
logic         memwrite;
logic [4:0]   rs1;
logic [4:0]   rs2;
logic [4:0]   rd;
logic [31:0]  immval;
logic         op_b_sel;
logic [1:0]   op_a_sel;
logic [1:0]   extend_sel;

//Reg_file 
logic           clk_reg;
logic           reset_reg;
logic           write_en;        //reg_write enable pin  
logic  [4:0]    rd_address;  //writeregister
logic  [4:0]    rs1_address;      //read reg1
logic  [4:0]    rs2_address;   //read reg2
logic [31:0]    rd_data;    //writedata
logic [31:0]    rs1_data;   //read data rs1
logic [31:0]    rs2_data;    //read data rs2

//Alu
logic  [5:0]    aluop;
logic [31:0]    operand_a;
logic [31:0]    operand_b;
logic [ 4:0]    shbits;
logic [31:0]    alu_out;
logic           branch;
logic [11:0]    aluout_addr;
//data_mem
logic           clk_d;
logic           reset_d;
logic [9:0]     addr_d;
logic [31:0]    data_in;
logic [31:0]    data_out;
logic           load_en;
logic           store_en;
logic           instruction;
logic           write_enable;

 pc pc_top(
.clk(clk)
,.reset(reset)
,.nextpc_sel(nextpc_sel)
,.branch(branch_i)
,.imm_i(imm_i)
,.pc_out(pc_out)
,.operand_a(operand_a));

assign addr_i = pc_out[9:0];

//INSTANTIATING ins_mem 
ins_mem ins_memtop(
 .clk(clk)
 ,.reset(reset)
 ,.write_en(write_en)
 ,.data_in(data_i)
 ,.addr_i(addr_i)
 ,.read_en(read_en_i)
 ,.data_o(data_o)
);
assign instruction = data_o;
//INSTANTIATING CONTROL unit 

control_unit control_unittop (
.ins(data_o)
,.pc(program_counter)
,.illegal(illegal)
,.memwrite(memwrite)
,.regwrite(regwrite)
,.memread(memread)
,.memtoreg(memtoreg)
,.branch(branch)
,.nextpc_sel(nextpc_sel)
,.rs1(rs1)
,.rs2(rs2)
,.rd(rd)
,.immval(immval)
,.aluop(aluop)
,.op_b_sel(op_b_sel)
//,.op_a_sel(op_a_sel)//
,.extend_sel(extend_sel)
);
 
assign operand_b = (op_b_sel==1) ? immval : rs2_data;
//assign write_en = regwrite;//

reg_file reg_filetop(
.clk(clk)
,.reset(reset)
,.write_en(regwrite)
,.rd_address(rd_address)
,.rs1_address(rs1_address) 
,.rs2_address(rs2_address)
,.rd_data(rd_data)
,.rs1_data(rs1_data)
,.rs2_data(rs2_data)
);

assign rd_data = (memtoreg == 1'b1) ? data_out: alu_out;


alu alutop(
.aluop(aluop)
,.operand_a(operand_a)
,.operand_b(operand_b)
,.shbits(shbits)
,.alu_out(alu_out)
,.branch(branch_i)
 
);


assign load_en= memread ;
//assign store_en= memwrite ;//

assign  aluout_addr= alu_out[13:2] ;

//INSTANTIATING DATA MEMORY
data_mem data_memtop(
 .clk(clk)
 ,.reset(reset)
 ,.store(write_en)
 ,.read_en(load_en)
 ,.data_in (data_in)
 ,.addr(aluout_addr)
 ,.data_out(data_out)
);
 

endmodule
