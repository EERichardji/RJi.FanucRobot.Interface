# RJi.FanucRobot.Interface

[English](README.md)

> 本库为基于 .NET Standard 2.0 和 .NET 8.0 的类库。

## ⚠️ 重要免责声明

1. 本库为**个人非官方第三方实现**，与 FANUC 官方无任何关联，并非官方 SDK 及官方配套工具。
2. 本内容仅供**个人学习、测试、技术研究**使用，**严禁直接用于工业现场、生产环境及正式设备**。
3. 作者不对本库的稳定性、兼容性、安全性做任何担保，因使用本库产生的一切设备故障、数据异常、经济损失等后果，均由使用者自行承担。

## 正文

FANUC 机器人 PC Interface 的 .NET 翻写库，基于 SNPX 协议实现，**无需安装 FRRJIf.dll 及官方依赖** 即可运行。

支持以下 FANUC 控制器（要求选件 **R651** 或 **R650+R553**）：

- **R-J3iB**：7D80/45+、7D81/09+、7D82/01+
- **R-J3iB Mate**：7D91/01+
- **R-30iA / R-30iA Mate**：全部版本
- **R-30iB / R-30iB Mate**：全部版本
- **R-30iB Plus / Mate Plus / Compact Plus / Mini Plus**：全部版本

**选件说明：**

- **R651（FRL Params）**：已内置 SNPX 通信支持，无需额外选件
- **R650（FRA Params）**：需同时选配 **R553（HMI Device SNPX）** 方可使用

## 安装

- **目标框架**：.NET Standard 2.0（兼容 .NET Framework 4.6.1+ / .NET Core 2.0+ / .NET 5+）
- **依赖**：无需第三方库；若使用依赖注入需 `Microsoft.Extensions.DependencyInjection`

## 快速开始

### 引入命名空间

```csharp
using RJi.FanucRobot.Interface
```

### 配置 RobotConnectionConfig

```csharp
// 默认配置（IP=127.0.0.1:60008）
var config = RobotConnectionConfig.Default;

// 自定义配置
var config = new RobotConnectionConfig
{
    IpAddress = "192.168.1.100",  // 机器人 IP
    Port = 60008,                 // SNPX 端口（默认 60008）
    ConnectionTimeoutMs = 10000,  // 连接超时（毫秒）
    ReadWriteTimeoutMs = 10000,   // 读写超时（毫秒）
    ClientId = 1024               // 客户端标识（首次连接时使用，默认 1024，通常无需修改）
};
```

### 创建客户端并连接

```csharp
// 方式一：默认配置（IP=127.0.0.1:60008）
using var robot = new FanucRobotClient();

// 方式二：自定义配置
using var robot = new FanucRobotClient(config);

// 使用构造函数传入的配置连接
bool connected = robot.Connect();              // 使用 config 中的 IpAddress + Port
bool connected = await robot.ConnectAsync();

// 或临时指定 IP 和端口（覆盖 config）
bool connected = robot.Connect("192.168.1.100");
bool connected = await robot.ConnectAsync("192.168.1.100", port: 60008);
```

### 断开连接

```csharp
robot.Disconnect();
```

### 依赖注入（MSDI）

```csharp
// IServiceCollection 注册方式
services.AddSingleton<IFanucRobotClient>(sp =>
{
    var config = new RobotConnectionConfig
    {
        IpAddress = "192.168.1.100",
        Port = 60008
    };
    var client = new FanucRobotClient(config);
    client.Connect();
    return client;
});

// 或使用工厂模式延迟连接
services.AddSingleton<IFanucRobotClient>(new FanucRobotClient(config));
// 在需要时手动调用 Connect()
```

## API 总览

所有 API 均提供同步 + 异步两种版本，异步方法以 `Async` 结尾。

---

### 数字信号（DigitalSignal）

| 属性        | 对应机器人信号 | 说明         |
| ----------- | -------------- | ------------ |
| `robot.DI`  | SDI            | 标准数字输入 |
| `robot.DO`  | SDO            | 标准数字输出 |
| `robot.RI`  | RDI            | 机器人输入   |
| `robot.RO`  | RDO            | 机器人输出   |
| `robot.UI`  | 用户面板输入   | UI           |
| `robot.UO`  | 用户面板输出   | UO           |
| `robot.SI`  | 系统输入       | SI           |
| `robot.SO`  | 系统输出       | SO           |
| `robot.WI`  | 焊接输入       | WI           |
| `robot.WO`  | 焊接输出       | WO           |
| `robot.WSI` | 焊接系统输入   | WSI          |
| `robot.WSO` | 焊接系统输出   | WSO          |

```csharp
// DI[1] 读取（标准数字输入第 1 路）
bool val = robot.DI.ReadSingle(1);
bool val = await robot.DI.ReadSingleAsync(1);

// DO[1]~DO[10] 批量读取
bool[] vals = robot.DO.Read(1, 10);
bool[] vals = await robot.DO.ReadAsync(1, 10);

// DO[1]=ON 写入（置位 DO[1]）
robot.DO.WriteSingle(1, true);
await robot.DO.WriteSingleAsync(1, true);

// DO[1]~DO[3] 批量写入（DO[1]=ON, DO[2]=OFF, DO[3]=ON）
robot.DO.Write(1, new[] { true, false, true });
await robot.DO.WriteAsync(1, new[] { true, false, true });
```

> RI/RO/UI/UO/SI/SO/WI/WO/WSI/WSO 提供完全相同的 ReadSingle/Read/WriteSingle/Write 及 Async 方法。

---

### 组信号（GroupSignal）

| 属性       | 对应机器人信号 | 说明             |
| ---------- | -------------- | ---------------- |
| `robot.GI` | GI             | 组输入（int 值） |
| `robot.GO` | GO             | 组输出（int 值） |

```csharp
// GI[1] 读取（组输入第 1 通道）
int val = robot.GI.ReadSingle(1);
int val = await robot.GI.ReadSingleAsync(1);

// GI[1]~GI[5] 批量读取
int[] vals = robot.GI.Read(1, 5);
int[] vals = await robot.GI.ReadAsync(1, 5);

// GO[1]=100 写入（组输出第 1 通道写入 100）
robot.GO.WriteSingle(1, 100);
await robot.GO.WriteSingleAsync(1, 100);

// GO[1]~GO[3] 批量写入（GO[1]=100, GO[2]=200, GO[3]=300）
robot.GO.Write(1, new[] { 100, 200, 300 });
await robot.GO.WriteAsync(1, new[] { 100, 200, 300 });
```

---

### 模拟信号（AnalogSignal）

| 属性       | 对应机器人信号 | 说明            |
| ---------- | -------------- | --------------- |
| `robot.AI` | AI             | 模拟输入（int） |
| `robot.AO` | AO             | 模拟输出（int） |

```csharp
// AI[1] 读取（模拟输入第 1 通道）
int ai = robot.AI.ReadSingle(1);
int ai = await robot.AI.ReadSingleAsync(1);

// AI[1]~AI[5] 批量读取
int[] ais = robot.AI.Read(1, 5);
int[] ais = await robot.AI.ReadAsync(1, 5);

// AO[1]=500 写入（模拟输出第 1 通道设 500）
robot.AO.WriteSingle(1, 500);
await robot.AO.WriteSingleAsync(1, 500);

// AO[1]~AO[3] 批量写入
robot.AO.Write(1, new[] { 100, 200, 300 });
await robot.AO.WriteAsync(1, new[] { 100, 200, 300 });
```

---

### PMC 信号（PmcSignal）

| 属性        | 说明                                                   |
| ----------- | ------------------------------------------------------ |
| `robot.Pmc` | PMC 信号访问（R 区继电器、K 区保持继电器、D 区数据表） |

```csharp
// R[1] 读取（PMC 内部继电器，位访问）
bool r = robot.Pmc.ReadRelay(1);
bool r = await robot.Pmc.ReadRelayAsync(1);

// R[1]=ON 写入
robot.Pmc.WriteRelay(1, true);
await robot.Pmc.WriteRelayAsync(1, true);

// K[1] 读取（PMC 保持继电器）
bool k = robot.Pmc.ReadKeep(1);
bool k = await robot.Pmc.ReadKeepAsync(1);

// K[1]=ON 写入
robot.Pmc.WriteKeep(1, true);
await robot.Pmc.WriteKeepAsync(1, true);

// D[1] 读取（PMC 数据表，字访问）
short d = robot.Pmc.ReadData(1);
short d = await robot.Pmc.ReadDataAsync(1);

// D[1]=12345 写入
robot.Pmc.WriteData(1, 12345);
await robot.Pmc.WriteDataAsync(1, 12345);
```

---

### 数值寄存器 R[i]

```csharp
// R[1] 读取（float 值）
float val = robot.NumReg.Read(1);
float val = await robot.NumReg.ReadAsync(1);

// R[1]=3.14 写入
robot.NumReg.Write(1, 3.14f);
await robot.NumReg.WriteAsync(1, 3.14f);

// R[1]~R[10] 批量读取
float[] vals = robot.NumReg.ReadBatch(1, 10);
float[] vals = await robot.NumReg.ReadBatchAsync(1, 10);

// R[1]~R[3] 批量写入
robot.NumReg.WriteBatch(1, new float[] { 1.1f, 2.2f, 3.3f });
await robot.NumReg.WriteBatchAsync(1, new float[] { 1.1f, 2.2f, 3.3f });
```

> R[i] 天然连续，`ReadBatchAsync` 内部一次性批量映射 + 批量 READ，性能最优。不需预注册。

---

### 位置寄存器 PR[i]

```csharp
// PR[1] 读取（Group 1，含关节坐标、笛卡尔坐标、配置字、刀具/用户坐标系）
PositionInfo pos = robot.PosReg.Read(1, group: 1);
PositionInfo pos = await robot.PosReg.ReadAsync(1, group: 1);

// PR[G2:1] 读取（Group 2，多运动组）
// ⚠ 如果机器人未配置 group 2，不会抛出异常，但返回全零数据
PositionInfo pos = robot.PosReg.Read(1, group: 2);
PositionInfo pos = await robot.PosReg.ReadAsync(1, group: 2);

// PR[1]~PR[5] 批量读取（Group 1）
PositionInfo[] posList = robot.PosReg.ReadBatch(1, 5);
PositionInfo[] posList = await robot.PosReg.ReadBatchAsync(1, 5);

// PR[1] 写入关节坐标（UF=1, UT=1）
var joint = new JointPosition { J1 = 0, J2 = -90, J3 = 0, J4 = 0, J5 = 0, J6 = 0 };
robot.PosReg.WriteJoint(1, joint, uf: 1, ut: 1);
await robot.PosReg.WriteJointAsync(1, joint, uf: 1, ut: 1);

// PR[2] 写入笛卡尔坐标（UF=1, UT=1）
var cart = new CartesianPosition { X = 100, Y = 200, Z = 300, W = 180, P = 0, R = 0 };
var cfg = new PositionConfig
{
    NonFFlip = PositionConfig.FlipState.NonFlip,
    DownUp = PositionConfig.ArmConfig.Down,
    BackTurn = PositionConfig.TurnConfig.Back,
    Turn1 = 0, Turn2 = 0, Turn3 = 0
};
robot.PosReg.WriteCartesian(2, cart, cfg, uf: 1, ut: 1);
await robot.PosReg.WriteCartesianAsync(2, cart, cfg, uf: 1, ut: 1);

// PR[1]~PR[3] 批量写入关节坐标（简化：同组同 UF/UT）
var joints = new[]
{
    new JointPosition { J1 = 0, J2 = -90, J3 = 0, J4 = 0, J5 = 0, J6 = 0 },
    new JointPosition { J1 = 30, J2 = -60, J3 = 30, J4 = 0, J5 = 90, J6 = 0 },
    new JointPosition { J1 = 60, J2 = -30, J3 = 60, J4 = 45, J5 = 45, J6 = 30 }
};
robot.PosReg.WriteJointBatch(1, joints, uf: 1, ut: 1);
await robot.PosReg.WriteJointBatchAsync(1, joints, uf: 1, ut: 1);

// PR[1]~PR[3] 批量写入关节坐标（完整：逐 PR 独立指定）
var ufArray = new short[] { 1, 2, 1 };   // 分别使用 UF1、UF2、UF1
var utArray = new short[] { 1, 1, 2 };   // 分别使用 UT1、UT1、UT2
var groups = new int[] { 1, 1, 2 };      // PR[1]、PR[2] 在组 1，PR[3] 在组 2
robot.PosReg.WriteJointBatch(1, joints, ufArray, utArray, groups);
await robot.PosReg.WriteJointBatchAsync(1, joints, ufArray, utArray, groups);

// PR[10]~PR[12] 批量写入笛卡尔坐标（简化：同组同 UF/UT）
var carts = new[]
{
    new CartesianPosition { X = 100, Y = 0, Z = 300, W = 0, P = 0, R = 0 },
    new CartesianPosition { X = 200, Y = 100, Z = 350, W = 90, P = 0, R = 0 },
    new CartesianPosition { X = 300, Y = 0, Z = 400, W = 180, P = 0, R = 0 }
};
var configs = new PositionConfig[]
{
    new PositionConfig
    {
        NonFFlip = PositionConfig.FlipState.NonFlip,
        DownUp = PositionConfig.ArmConfig.Down,
        BackTurn = PositionConfig.TurnConfig.Back,
        Turn1 = 0, Turn2 = 0, Turn3 = 0
    },
    new PositionConfig
    {
        NonFFlip = PositionConfig.FlipState.NonFlip,
        DownUp = PositionConfig.ArmConfig.Down,
        BackTurn = PositionConfig.TurnConfig.Back,
        Turn1 = 0, Turn2 = 0, Turn3 = 0
    },
    new PositionConfig
    {
        NonFFlip = PositionConfig.FlipState.NonFlip,
        DownUp = PositionConfig.ArmConfig.Down,
        BackTurn = PositionConfig.TurnConfig.Back,
        Turn1 = 0, Turn2 = 0, Turn3 = 0
    }
};
robot.PosReg.WriteCartesianBatch(10, carts, configs, uf: 1, ut: 1);
await robot.PosReg.WriteCartesianBatchAsync(10, carts, configs, uf: 1, ut: 1);

// PR[10]~PR[12] 批量写入笛卡尔坐标（完整：逐 PR 独立指定）
robot.PosReg.WriteCartesianBatch(10, carts, configs, ufArray, utArray, groups);
await robot.PosReg.WriteCartesianBatchAsync(10, carts, configs, ufArray, utArray, groups);
```

**`PositionConfig` 属性说明：**

| 属性        | 类型                        | 枚举值                 | 机器人配置字符串含义                   |
| ----------- | --------------------------- | ---------------------- | -------------------------------------- |
| `NonFFlip`  | `PositionConfig.FlipState`  | `NonFlip=0` / `Flip=1` | F=无翻转, FL=翻转                      |
| `LeftRight` | `PositionConfig.HandConfig` | `Left=0` / `Right=1`   | L=左手系, R=右手系（**示教器不显示**） |
| `DownUp`    | `PositionConfig.ArmConfig`  | `Down=0` / `Up=1`      | D=肘下垂, U=肘上仰                     |
| `BackTurn`  | `PositionConfig.TurnConfig` | `Back=0` / `Turn=1`    | B=后侧(Back), T=前侧(Turn)             |
| `Turn1`     | `short`                     | 0~7                    | 关节 J1 回转圈数                       |
| `Turn2`     | `short`                     | 0~7                    | 关节 J2 回转圈数                       |
| `Turn3`     | `short`                     | 0~7                    | 关节 J3 回转圈数                       |

> FANUC 配置字符串格式如 `[F, L, D, B, 0, 0, 0]`，对应 `NonFlip, Left, Down, Back, Turn1=0, Turn2=0, Turn3=0`。

---

### 字符串寄存器 SR[i]

```csharp
// SR[1] 读取
string str = robot.StrReg.Read(1);
string str = await robot.StrReg.ReadAsync(1);

// SR[1]="HELLO" 写入
robot.StrReg.Write(1, "HELLO");
await robot.StrReg.WriteAsync(1, "HELLO");

// SR[1]~SR[10] 批量读取
string[] strs = robot.StrReg.ReadBatch(1, 10);
string[] strs = await robot.StrReg.ReadBatchAsync(1, 10);

// SR[1]~SR[3] 批量写入
robot.StrReg.WriteBatch(1, new[] { "HELLO", "WORLD", "TEST" });
await robot.StrReg.WriteBatchAsync(1, new[] { "HELLO", "WORLD", "TEST" });
```

---

### 标志寄存器 F[i]

```csharp
// F[1] 读取（0/1）
bool flag = robot.Flag.Read(1);
bool flag = await robot.Flag.ReadAsync(1);

// F[1]=ON 写入
robot.Flag.Write(1, true);
await robot.Flag.WriteAsync(1, true);

// F[1]~F[20] 批量读取
bool[] flags = robot.Flag.ReadBatch(1, 20);
bool[] flags = await robot.Flag.ReadBatchAsync(1, 20);

// F[1]~F[5] 批量写入（F[1]=ON, F[2]=OFF, F[3]=ON, F[4]=OFF, F[5]=ON）
robot.Flag.WriteBatch(1, new[] { true, false, true, false, true });
await robot.Flag.WriteBatchAsync(1, new[] { true, false, true, false, true });
```

---

### 系统变量

```csharp
// 支持类型：
//   INT（4 字节整数）、REAL（float）、BOOL（布尔）、STRING（字符串）、POS（位置）

// ⚠ 如果变量名不存在或拼写错误，不会抛出异常，返回全零/空数据
// 使用前请确认变量名正确，并对返回值进行合理性校验

// $MOR_GRP[1].$ANGTOL[1] 读取（整数）
int intVal = robot.SystemVariables.ReadInt("$MOR_GRP[1].$ANGTOL[1]");
int intVal = await robot.SystemVariables.ReadIntAsync("$MOR_GRP[1].$ANGTOL[1]");

// $SCRN_OP[1].$MAX_RPM 读取（浮点数）
float realVal = robot.SystemVariables.ReadFloat("$SCRN_OP[1].$MAX_RPM");
float realVal = await robot.SystemVariables.ReadFloatAsync("$SCRN_OP[1].$MAX_RPM");

// $SS_ENB 读取（布尔）
bool boolVal = robot.SystemVariables.ReadBool("$SS_ENB");
bool boolVal = await robot.SystemVariables.ReadBoolAsync("$SS_ENB");

// $SYSNAME 读取（字符串）
string strVal = robot.SystemVariables.ReadString("$SYSNAME");
string strVal = await robot.SystemVariables.ReadStringAsync("$SYSNAME");

// $MNUFRAME[1] 读取（位置变量）
PositionInfo posVal = robot.SystemVariables.ReadPosition("$MNUFRAME[1]");
PositionInfo posVal = await robot.SystemVariables.ReadPositionAsync("$MNUFRAME[1]");

// $MOR_GRP[1].$ANGTOL[1]=5 写入
robot.SystemVariables.WriteInt("$MOR_GRP[1].$ANGTOL[1]", 5);
await robot.SystemVariables.WriteIntAsync("$MOR_GRP[1].$ANGTOL[1]", 5);

// 批量变量组（预注册 → 一次性读取，性能最优）
var group = robot.SystemVariables.CreateVariableGroup(new List<(string, VariableType)>
{
    ("$MOR_GRP[1].$ANGTOL[1]", VariableType.INT),
    ("$SCRN_OP[1].$MAX_RPM",   VariableType.REAL),
    ("$SYSNAME",                VariableType.STRING)
});

// 全量读取（1 次 READ 交互，适合循环监控）
object[] batch = group.Read();
object[] batch = await group.ReadAsync();

// 全量写入
group.Write(new object[] { 100, 2000.0f, "Hello" });
await group.WriteAsync(new object[] { 100, 2000.0f, "Hello" });
```

---

### 当前位置

```csharp
// 世界坐标系笛卡尔坐标（Grop 1）
CartesianPosition cart = robot.Position.ReadWorldPosition(group: 1);
CartesianPosition cart = await robot.Position.ReadWorldPositionAsync(group: 1);

// 关节坐标（Grop 1）
JointPosition joint = robot.Position.ReadJointPosition(group: 1);
JointPosition joint = await robot.Position.ReadJointPositionAsync(group: 1);

// 指定 UF 下的完整位置（含关节 + 笛卡尔 + 配置）
PositionInfo pos = robot.Position.ReadUserPosition(ufNumber: 1, group: 1);
PositionInfo pos = await robot.Position.ReadUserPositionAsync(ufNumber: 1, group: 1);
```

---

### 任务状态（TaskManager）

```csharp
// PRG[1] 监控任务 1 的状态
TaskInfo task = robot.Task.Read(index: 1);
TaskInfo task = await robot.Task.ReadAsync(index: 1);

// 忽略 Karel 任务（PRG[K1]）
TaskInfo task = await robot.Task.ReadAsync(index: 1, type: TaskType.IgnoreKarel);
```

**TaskType 枚举：**

| 枚举值             | SETASG 格式 | 说明                |
| ------------------ | ----------- | ------------------- |
| `Normal`           | PRG[i]      | 标准任务监控        |
| `IgnoreKarel`      | PRG[K{i}]   | 忽略 Karel 任务     |
| `IgnoreMacro`      | PRG[M{i}]   | 忽略宏任务          |
| `IgnoreMacroKarel` | PRG[MK{i}]  | 忽略 Karel 和宏任务 |

**TaskInfo 属性：**

| 属性             | 类型   | 说明                                         |
| ---------------- | ------ | -------------------------------------------- |
| `ProgName`       | string | 当前运行的程序名                             |
| `LineNumber`     | int    | 当前运行行号                                 |
| `State`          | int    | 任务状态码（1=运行, 2=暂停, 3=停止, 4=异常） |
| `StateText`      | string | 状态描述（RUN / PAUSE / STOP / ERROR）       |
| `ParentProgName` | string | 父程序名（子程序调用时有效）                 |

---

### 报警管理（AlarmManager）

```csharp
// E1 报警历史（List 模式、Full 格式，前 10 条）
AlarmItem[] alarms = robot.Alarm.Read(count: 10);
AlarmItem[] alarms = await robot.Alarm.ReadAsync(count: 10);

// 当前报警（Current 模式、Short 格式）
AlarmItem[] current = await robot.Alarm.ReadAsync(
    count: 10,
    type: AlarmType.Current,
    mode: AlarmMessageMode.Short
);

// 清除报警（type=0 表示清除当前报警）
robot.ClearAlarm(type: 0);
await robot.ClearAlarmAsync(type: 0);
```

**AlarmType 枚举：**

| 枚举值     | SETASG 格式 | 说明                 |
| ---------- | ----------- | -------------------- |
| `List`     | ALM[E1]     | 报警历史列表（默认） |
| `Current`  | ALM[1]      | 当前报警             |
| `Password` | ALM[P1]     | 密码报警             |

**AlarmMessageMode 枚举：**

| 枚举值       | 字/条 | 说明                                                     |
| ------------ | ----- | -------------------------------------------------------- |
| `Full` (0)   | 100   | 完整：AlarmMessage + CauseAlarmMessage + SeverityMessage |
| `Short` (1)  | 51    | 仅基本字段 + AlarmMessage                                |
| `Medium` (2) | 91    | 基本字段 + AlarmMessage + CauseAlarmMessage              |

**AlarmItem 属性：**

| 属性                | 类型      | 说明                                  |
| ------------------- | --------- | ------------------------------------- |
| `AlarmId`           | short     | 报警 ID（同一位置多次报警的累计编号） |
| `AlarmNumber`       | short     | 报警编号（控制器定义的报警代码）      |
| `CauseAlarmId`      | short     | 原因报警 ID（0=无）                   |
| `CauseAlarmNumber`  | short     | 原因报警编号                          |
| `Severity`          | short     | 严重级别（原始代码，对照示教器解读）  |
| `AlarmMessage`      | string    | 报警消息文本（Short 模式下为空）      |
| `CauseAlarmMessage` | string    | 原因报警消息（Medium/Full 模式）      |
| `SeverityMessage`   | string    | 严重级别描述（仅 Full 模式）          |
| `Timestamp`         | DateTime? | 报警时间（合并年月日时分秒）          |

---

### 注释读取（CommentManager）

```csharp
// 通过枚举读取 SI[C1]（系统输入第 1 路注释）
string comment = robot.Comment.Read(CommentType.SI, index: 1);
string comment = await robot.Comment.ReadAsync(CommentType.SI, index: 1);

// 通过字符串前缀读取
string comment = robot.Comment.Read("DI", 1);
string comment = await robot.Comment.ReadAsync("SR", 1);

// 通过枚举写入 DO[C1]（数字输出第 1 路注释）
robot.Comment.Write(CommentType.DO, index: 1, value: "抓取工位");
await robot.Comment.WriteAsync(CommentType.DO, index: 1, value: "抓取工位");

// 通过字符串前缀写入
robot.Comment.Write("R", 5, "温度阈值");
await robot.Comment.WriteAsync("SR", 1, "产品名称");
```

**CommentType 枚举（共 20 种）：**

信号注释：`DI` `DO` `RI` `RO` `UI` `UO` `SI` `SO` `GI` `GO` `AI` `AO` `WI` `WO` `WSI` `WSO`
寄存器注释：`SR` `R` `PR`
标志注释：`F`

SNPX 格式示例：`SI[C1]` = SI 信号第 1 路注释，`R[C2]` = R 寄存器第 2 路注释，`F[C3]` = F 标志第 3 路注释。

---

### ClearRAssignment

```csharp
// 清除控制器上所有 SETASG 的 %R 寄存器映射
// ⚠ 极少使用：首次连接时内部自动调用 CLRASG
// 仅在重置 DataTable 或切换数据场景时手动调用
robot.ClearRAssignment();
await robot.ClearRAssignmentAsync();
```

### SetClientId

```csharp
// 设置 SNPX 客户端标识
// ⚠ 默认值 1024，首次连接时已使用 config.ClientId
// 通常不需要主动调用
robot.SetClientId(1024);
```

---

### 扩展方法

```csharp
using RJi.FanucRobot.Interface.Common.Extensions;

// float 扩展
float rounded = 3.14159f.RoundTo(3);         // → 3.142
float[] arr = new[] { 1.234f, 5.678f }.RoundAll(2);

// 关节坐标近似比较（容差 1e-3）
bool approx = pos.Joint.IsApproximately(otherJoint, epsilon: 1e-3f);

// 关节坐标是否全零
bool isZero = pos.Joint.IsZero();

// 格式化为显示字符串 "J1=0.000 J2=-30.000 ..."
string display = pos.Joint.ToDisplayString(decimals: 3);

// 关节坐标深拷贝
var clone = pos.Joint.Clone();

// 关节坐标保留小数位数
var rounded = pos.Joint.RoundTo(decimals: 4);

// 计算两个关节坐标的差值绝对值
var diff = pos.Joint.Diff(otherJoint);

// 笛卡尔坐标近似比较
bool approx = pos.Cartesian.IsApproximately(otherCart);

// 笛卡尔坐标格式化 "X=100.000 Y=200.000 Z=300.000 ..."
string display = pos.Cartesian.ToDisplayString(decimals: 3);

// 笛卡尔坐标深拷贝 / 保留小数 / 差值
var clone = pos.Cartesian.Clone();
var rounded = pos.Cartesian.RoundTo(3);
var diff = pos.Cartesian.Diff(otherCart);
```

## 重要说明

### SNPX 串行协议

底层基于 SNPX 协议通信，**同一连接不支持并发读写**。本库内部已通过 `SemaphoreSlim` 严格串行化所有请求，同实例的多线程调用无意义——请求会自动排队串行执行。

- 批量读取大量数据时建议使用异步 API，避免阻塞调用线程
- 同一 PC 连接同一机器人不能并发读取（串行协议限制）

### 并发连接数

机器人控制器版本决定同一机器人可同时接受的 PC 连接上限：

- **7DA3/06+ 及 7DA4+**：最多 **4 个** 并发 PC 连接
- **7DA3/05 及更早**：仅 **1 个** PC 连接

每个连接需使用不同的 `ClientId`（`config.ClientId`），控制器按 ID 区分会话。超过上限的连接请求会失败。

### 数据读取限制

官方库受 DataTable 容量限制，每次 `AddXXX` 和 `Refresh` 管理繁琐。本库通过 **三级缓存策略** 突破限制：

1. **缓存命中**：已注册的地址直接 READ（1 次交互）
2. **永久分配**：RegisterMap 空间充足时分配 + SETASG（2 次交互，后续永久缓存）
3. **临时槽位**：空间不足时通过中央临时槽位 SETASG + READ（每次 2 次交互）

理论上可无限制读取任意数量变量/寄存器，但 SNPX 特性决定变量越多读取时间越长。批量读取可显著减少交互次数。**建议大批量读取时使用异步 API 防止阻塞调用线程**。

### SNPX 协议静默失败

SNPX 协议本身不提供操作结果的反馈机制——SETASG 映射是否成功、系统变量是否存在、多运动组是否配置，**机器人控制器都不会返回明确的成功/失败状态**。本库将收到的数据直接返回给调用方，不会因为"数据似乎无效"而主动抛出异常。

**典型静默失败场景：**

| 场景                                   | 表现                            | 说明                                     |
| -------------------------------------- | ------------------------------- | ---------------------------------------- |
| 读取不存在的系统变量                   | 返回全零/空数据，**无异常**     | 机器人没有该变量时，%R 寄存器保持初始值  |
| 写入不存在的系统变量                   | 写入方法返回 `true`，**无异常** | 命令已被接收但被机器人静默忽略           |
| `group > 1` 读取位置寄存器但该组未配置 | 返回全零坐标，**无异常**        | 机器人没有该运动组时，SETASG 映射不生效  |
| `group > 1` 写入位置寄存器但该组未配置 | 写入方法返回 `true`，**无异常** | 数据写入 %R 寄存器但并未实际映射到 PR    |
| 读取超出范围的寄存器索引               | 返回全零/空数据，**无异常**     | 机器人不校验索引范围，直接返回 %R 初始值 |

**使用建议：**

```csharp
// 示例：读取后主动验证数据有效性
var pos = await robot.PosReg.ReadAsync(1, group: 2);

// 检查关节坐标是否全零（全零通常表示映射未生效）
if (pos.Joint.IsZero())
{
    // 可能是 group 2 未配置，或 PR[G2:1] 不存在
    Console.WriteLine("警告：读取到的位置数据全零，请确认机器人运动组配置正确");
}

// 系统变量同理
var val = await robot.SystemVariables.ReadFloatAsync("$NON_EXIST_VAR");
if (Math.Abs(val) < float.Epsilon)
{
    Console.WriteLine("警告：系统变量可能不存在，请确认变量名正确");
}
```

> **核心原则：没有异常不代表操作成功。** 使用方应根据业务逻辑对返回数据进行合理性校验（如非零检查、范围检查、与预期值对比等），不要仅凭无异常判定操作生效。

### 错误处理

```csharp
try
{
    float val = await robot.NumReg.ReadAsync(1);
}
catch (RobotException ex)
{
    Console.WriteLine($"[{ex.ErrorCode}] {ex.Message}");
    // 连接错误需销毁客户端并重建
    robot.Dispose();
}
```

| 错误码                                                         | 说明           |
| -------------------------------------------------------------- | -------------- |
| `NotConnected`                                                 | 未连接到机器人 |
| `ConnectionFailed` / `ConnectionTimeout` / `ConnectionRefused` | 连接失败       |
| `ProtocolError` / `InvalidResponse`                            | 协议响应异常   |
| `ReadError` / `WriteError`                                     | 读写错误       |
| `InvalidAddress` / `InvalidData`                               | 参数错误       |

## 与官方库 API 对比

| 功能             | 官方库（FRRJIf.dll）                                    | 本库                                                         |
| ---------------- | ------------------------------------------------------- | ------------------------------------------------------------ |
| **依赖**         | 需安装 FRRJIf.dll，仅 Windows                           | 纯 .NET Standard 2.0，跨平台                                 |
| **连接**         | `Core.Connect(hostName)`                                | `robot.Connect(ip, port)` / 无参重载                         |
| **断开**         | `Core.Disconnect()`                                     | `robot.Disconnect()`                                         |
| **数字信号**     | `ReadSdo/Sdi/Rdo/Rdi/So/Si/Uo/Ui` 传 `Array` + `Count`  | `robot.DI.ReadSingle/Read` 直接返回 `bool`/`bool[]`          |
| **组信号**       | `ReadGo/Gi/WriteGo/Gi` 传 `Array`                       | `robot.GI.Read/Write` 直接返回 `short[]`                     |
| **模拟信号**     | 通过 GI/GO 索引 +1000 间接访问                          | `robot.AI/AO.ReadSingle/WriteSingle` 直接读写                |
| **PMC 信号**     | `ReadSdo(Index≥11001)` + `ReadPmcr2`                    | `robot.Pmc.ReadRelay/ReadKeep/ReadData` 分类访问             |
| **数值寄存器**   | `AddNumReg → GetValue/SetValues`（需 DataTable 预注册） | `robot.NumReg.Read/Write/ReadBatch` 即用即读                 |
| **位置寄存器**   | `AddPosReg` 或 `AddPosRegXyzwpr` + `Refresh`            | `robot.PosReg.Read/WriteJoint/WriteCartesian`                |
| **字符串寄存器** | `AddSysVar(STRREG)` 间接访问                            | `robot.StrReg.Read/Write/ReadBatch`                          |
| **标志寄存器**   | `AddFlag` + `Refresh`                                   | `robot.Flag.Read/Write/ReadBatch`                            |
| **系统变量**     | `AddSysVar/AddSysVarPos` + `Refresh`（需逐条注册）      | `robot.SystemVariables.ReadXxx/WriteXxx` 直接读写            |
| **系统变量批量** | 需逐条 AddSysVar，受 DataTable 容量限制                 | `CreateVariableGroup` 一次注册、批量读写                     |
| **当前位置**     | `AddCurPosUF` + `Refresh`                               | `robot.Position.ReadWorldPosition/ReadJointPosition`         |
| **任务监控**     | `AddTask` + `Refresh`                                   | `robot.Task.Read` 支持多种 TaskType                          |
| **报警管理**     | `AddAlarm` + `GetValue`（需指定类型和数量）             | `robot.Alarm.Read` 支持 AlarmType + AlarmMessageMode         |
| **注释读写**     | `AddSysVar(XX_COMMENT)` 间接访问                        | `robot.Comment.Read/Write` 支持 CommentType 枚举或字符串前缀 |
| **清除报警**     | `ClearAlarm(type)`                                      | `robot.ClearAlarm/ClearAlarmAsync`                           |
| **容量限制**     | DataTable 固定大小，超限需新建 DataTable                | 三级缓存自动管理，无理论上限                                 |
| **数据管理**     | 需手动 AddXXX → Refresh → GetValue                      | 自动 SETASG + 缓存，即用即读                                 |
| **同步版本**     | 仅有同步                                                | 同步 + 异步 (`Async` 后缀)                                   |
| **依赖注入**     | 不支持                                                  | 支持 MSDI 注册                                               |
