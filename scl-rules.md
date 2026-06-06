# TIA Portal SCL 规则

本文件用于生成、修改、审查 `.scl` 源文件。

## 总原则

写 SCL 时必须做到：

- 块接口完整。
- 变量声明完整。
- 局部/接口变量使用 `#` 前缀。
- `.scl` 文件保存为 UTF-8 BOM。
- FB 多实例赋值后显式调用。
- 状态机有默认分支和复位路径。
- 输出导入、编译、仿真步骤。

## `#` 前缀规则

在 TIA Portal SCL 中，当前块的接口变量、局部变量、静态变量、多实例通常用 `#` 访问。

### 需要 `#` 的对象

```scl
#In_Start       // VAR_INPUT
#Out_Run        // VAR_OUTPUT
#Io_Data        // VAR_IN_OUT
#Stat_State     // VAR / Static
#Temp_Index     // VAR_TEMP
#instMotor      // 当前 FB 中声明的多实例
```

正确示例：

```scl
#Out_Run := #In_Start AND NOT #In_Stop;
#Temp_Value := #In_Value + 1;
```

错误示例：

```scl
Out_Run := In_Start AND NOT In_Stop; // 错：局部/接口变量缺少 #
```

### 通常不加 `#` 的对象

```scl
"GlobalDB".SpeedSetpoint := 100.0; // 全局 DB
"MotorTag" := TRUE;                // PLC 全局标签，视命名而定
%Q0.0 := TRUE;                      // 绝对地址
```

## UTF-8 BOM 规则

给用户生成或修改 `.scl` 文件，必须用 UTF-8 BOM。
这样可避免博图导入时中文注释、字符串、符号乱码。

### Python 写文件

```bash
python3 << 'PY'
from pathlib import Path
content = '''FUNCTION_BLOCK "FB_Example"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1

VAR_INPUT
    In_Start : Bool; // 启动命令
END_VAR

VAR_OUTPUT
    Out_Run : Bool; // 运行状态
END_VAR

BEGIN
    #Out_Run := #In_Start;
END_FUNCTION_BLOCK
'''
Path('example.scl').write_text(content, encoding='utf-8-sig')  # content 在上方已定义
PY
```

### 批量转换目录中的 `.scl`

```bash
python3 << 'PY'
from pathlib import Path
for p in Path('.').glob('*.scl'):
    text = p.read_text(encoding='utf-8-sig')
    p.write_text(text, encoding='utf-8-sig')
    print('OK', p)
PY
```

### 检查 BOM

```bash
xxd -l 3 example.scl
# 期望前三字节：ef bb bf
```

## FB 多实例规则

在一个 FB 内声明另一个 FB 的实例，称为多实例。
核心铁律：

```text
先给接口赋值 → 再调用实例 → 再读取输出
```

### 分行调用写法

```scl
#opt.InValue := #SomeValue;
#opt.Enable := TRUE;
#opt();
#Result := #opt.OutValue;
```

### 参数化调用写法

```scl
#opt(
    InValue := #SomeValue,
    Enable := TRUE,
    OutValue => #Result
);
```

### 错误写法

```scl
#opt.InValue := #SomeValue;
#Result := #opt.OutValue; // 错：未调用 #opt()，可能读到上一周期输出
```

### 审查点

- 多实例是否在 `VAR` 静态区声明。
- 是否在读取输出前实际调用。
- 调用顺序是否符合依赖关系。
- 同一扫描周期内是否重复调用导致状态混乱。

## 变量声明模板

```scl
FUNCTION_BLOCK "FB_Device"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1

VAR_INPUT
    In_Start : Bool;   // 启动命令
    In_Stop  : Bool;   // 停止命令
    In_Reset : Bool;   // 复位命令
END_VAR

VAR_OUTPUT
    Out_Run  : Bool;   // 运行状态
    Out_Done : Bool;   // 完成状态
    Out_Alm  : Bool;   // 报警状态
END_VAR

VAR
    Stat_State : Int;  // 状态机
    Stat_LastStart : Bool; // 启动上一周期
END_VAR

VAR_TEMP
    Temp_StartRise : Bool; // 启动上升沿
END_VAR

BEGIN
    #Temp_StartRise := #In_Start AND NOT #Stat_LastStart;
    #Stat_LastStart := #In_Start;

    IF #In_Reset THEN
        #Stat_State := 0;
        #Out_Run := FALSE;
        #Out_Done := FALSE;
        #Out_Alm := FALSE;
    ELSE
        CASE #Stat_State OF
            0:
                #Out_Run := FALSE;
                #Out_Done := FALSE;
                IF #Temp_StartRise THEN
                    #Stat_State := 10;
                END_IF;

            10:
                #Out_Run := TRUE;
                IF #In_Stop THEN
                    #Stat_State := 0;
                END_IF;

            ELSE
                #Stat_State := 0;
        END_CASE;
    END_IF;
END_FUNCTION_BLOCK
```

## FC 纯计算模板

```scl
FUNCTION "FC_Scale" : Real
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1

VAR_INPUT
    In_Raw : Int;      // 原始值
    In_Min : Real;     // 工程下限
    In_Max : Real;     // 工程上限
END_VAR

VAR_TEMP
    Temp_Ratio : Real;
END_VAR

BEGIN
    #Temp_Ratio := INT_TO_REAL(#In_Raw) / 27648.0;
    "FC_Scale" := #In_Min + (#In_Max - #In_Min) * #Temp_Ratio;
END_FUNCTION
```

## UDT 模板

```scl
TYPE "UDT_Device"
VERSION : 0.1
STRUCT
    Cmd_Start : Bool;  // 启动命令
    Cmd_Stop  : Bool;  // 停止命令
    Sts_Run   : Bool;  // 运行状态
    Alm_Fault : Bool;  // 故障报警
END_STRUCT;
END_TYPE
```

## 命名建议

| 类别 | 前缀 | 示例 |
|---|---|---|
| 输入 | `In_` | `In_Start` |
| 输出 | `Out_` | `Out_Run` |
| 输入输出 | `Io_` | `Io_Data` |
| 静态 | `Stat_` | `Stat_State` |
| 临时 | `Temp_` | `Temp_Index` |
| 命令 | `Cmd_` | `Cmd_Start` |
| 状态 | `Sts_` | `Sts_Ready` |
| 报警 | `Alm_` | `Alm_Timeout` |

## 导入源文件步骤

```text
项目树 → PLC → 外部源文件 → 添加新的外部文件 → 选择 .scl → 从源生成块 → 编译
```

## 交付代码时的输出

```text
## 已生成/修改
- 文件：[路径]
- 块：[FB/FC/DB 名称]

## 关键逻辑
- [状态机/算法/互锁说明]

## 导入步骤
1. 外部源文件添加 .scl
2. 从源生成块
3. 软件全部重建
4. 下载到 PLCSIM 或 PLC

## 检查点
- UTF-8 BOM
- # 前缀
- FB 多实例调用
- CPU/版本兼容性
```
