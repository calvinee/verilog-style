# Verilog代码风格

## 1. 一般规则
* 缩进宽度：4（转换为空格）。
* 对齐规则：```begin, (, [, {```和对应语句处于同一行。
* 二元运算符前后、```if```等关键字后、```begin```前用空格分隔。
* 变量命名：在明确含义的情况下尽量简洁。
* 注释：


## 2. 文件头
以注释的形式对文件的内容进行简要描述，尽量简洁。
```verilog
//=============================================================================
// File Name    : {文件名}
// Created On   : {创建时间 yyyy-mm-dd hh:mm:ss}
// Last Modified: {更新日期 yyyy-mm-dd hh:mm:ss}
// Author       : {作者}
// Description  : {功能描述}
//=============================================================================
```
除头文件外，所有模块定义``` `timescale```。

## 3. 宏定义
宏定义统一采用大写加下划线的形式命名，在宏定义前给出相应的注释说明变量含义。对于有单位的变量（如时间、频率等），建议在变量名结尾加上单位。
```verilog
// 系统统一用时钟上升沿
`define CLK_EDGE posedge clk
// DDR接口位宽：256bit
`define DDR_BW 256
// 时钟频率：100MHz
`define CLK_FREQ_MHZ 100
```

## 4. 模块及端口声明
一个文件只允许包含一个模块的定义，文件名应和模块名称一致。模块的命名采用小写字母和下划线组合的形式，命名应尽量简洁。不要将模块名加入到端口名称中。

模块的参数采用大写加下划线命名，命名风格同宏定义。端口参数只允许以下两类：1. 端口位宽设置时需要用到的参数；2. 外部可配置的参数。对于端口需要用到的，但外部无法配置的参数，必须在参数注释中注明不可配置。通常这会是由可配参数推导出的参数。

模块端口采用小写字符加下划线的方式。端口名称需要表明信号含义及方向，对于有明确协议的一组信号，采用```{类型}_{协议名}_```作为前缀。对于除时钟和复位信号以外的一般信号，采用```i_```或```o_```作为前缀表示输入或输出。信号的位宽均采用```[{位宽}-1:0]```的描述方式。应对接口信号及参数做必要的注释。对于成组的接口可以在一组信号前统一写注释。不要求所有接口信号名称对齐，但每一组信号应对齐名称。包含位宽定义的信号应对齐左右中括号。只有采用```tri```形式的端口时才在声明中加入类型。

时钟统一命名为```clk```。复位信号命名为```rst```，异步复位信号在名称上加前缀```a```，低电平有效复位加后缀```_n```。包含时钟和复位信号的模块应将两者置于模块端口最前列。包含多组复位和时钟信号的模块，应首先将不同的时钟和复位信号对应的端口信号分组放置，然后将对应时钟及复位信号放在每一组的最前列。

```verilog
module adder_tree#(
    parameter DATA_IN_W = 8,    // 输入数据位宽
    parameter DATA_NUM  = 16,   // 输入数据个数
    parameter DATA_O_W  = DATA_IN_W + $clog2(DATA_NUM)
                                // 输出数据位宽（不可配置）
    )(
    input   clk,
    input   rst_n,

    input   i_config,       // 配置脉冲，高电平有效
    input   i_round_type,   // 输出舍入类型，0: 截断，1：四舍五入

    // 输入数据，axi_stream slave接口
    input   [DATA_IN_W * DATA_NUM   -1 : 0] s_axis_din_data,
    input                                   s_axis_din_valid,
    output                                  s_axis_din_ready,

    // 输出数据，axi_stream master接口
    output  [DATA_O_W               -1 : 0] m_axis_dout_data,
    output                                  m_axis_dout_valid,
    input                                   m_axis_dout_ready,
    );

    /* 模块定义... */

endmodule
```

## 5. 逻辑代码

### 5.1. 变量声明

变量的命名采用小写字母和下划线组合的形式。对于低电平有效的信号，应采用```_n```后缀，对于寄存器变量（注意不是指reg类型），应采用```_r```后缀。对于单纯为了延时而使用的寄存器变量，也可以采用```_d{x}```表示，x应从0开始计数。当仅有一个延时时可以不写x以表示d0。采用数组移位进行延时操作时，应从0开始依次向高位移位，以确保```_d{x}```和```_d[x]```的意义一致。不包含后缀的变量默认为组合逻辑的输出。

### 5.2. 例化模块

例化模块时模块实例名称采用```u_{模块名}_{序号}```的方式，或者```u_{模块名}_{实际功能}```的方式。应保证所有可以配置的参数被配置，且所有端口被连接。悬空的端口也应该以```/* not_used */```填入端口。模块例化时每个端口及其连接独立占据一行，顺序同模块声明时的顺序一致。

子模块的端口如果和顶层端口直接连接，则无须声明中间变量，直接连接即可。对于子模块之间的信号连接，中间变量的命名应综合考虑两边端口的名称，去除端口模式（i,o,m,s等），添加连接两端的名称，保留端口功能。可以根据实际情况进行命名。

同一个模块需要例化多个的时候，应采用generate块的形式，明确接口信号的连接方式。不要采用模块数组的形式，避免接口信号自动连接出现错误。

### 5.3. 局部参数

模块内部参数采用```localparam```关键字声明，命名采用大写字母和下划线组合的形式。涉及直接赋值的参数应标明位宽。

### 5.4. 代码块

基本规则：相关的逻辑连同变量声明尽量放在一起。可区分的逻辑块可以用注释分隔，注明每一段代码的大致功能。相似行为（具有相同赋值条件）的寄存器可以合并到一个```always```块中实现，不相似的在不同的```always```块中实现。

组合逻辑：一般采用```assign```。需要使用```always```时，为防止敏感表疏漏，统一采用```always @ (*)```

状态机：采用三段格式（状态定义+状态跳转+次态逻辑）

```verilog
localparam S_IDLE   = 2'b00;    // 状态机状态编码
localparam S_BUSY   = 2'b01;
localparam S_DONE   = 2'b10;

reg     [1 : 0] state_r;        // 当前状态
wire    [1 : 0] state_nxt;      // 次态

always @ (posedge clk) begin    // 状态跳转
    if (~rst_n) begin
        state_r <= S_IDLE;
    end
    else begin
        state_r <= state_nxt;
    end
end

always @ (*) begin              // 次态逻辑
    case(state_r)
    S_IDLE: begin
        state_nxt = ...
    end
    S_BUSY: ...
    S_DONE: ...
    default: ...
    endcase
end
```

逻辑表达式：较长的逻辑表达式可以通过换行的方式增加可读性

## 设计规范