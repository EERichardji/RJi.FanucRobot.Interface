# RJi.FanucRobot.Interface  
[简体中文](README.zh.md)
> This library is based on .NET Standard 2.0 and .NET 8.0.

## ⚠️ Important Disclaimer

1. This library is an **unofficial third-party implementation** by an individual, with no affiliation to FANUC. It is NOT an official SDK or supporting tool.
2. This content is intended **solely for personal learning, testing, and technical research**. It is **strictly prohibited from use in industrial environments, production systems, or formal equipment**.
3. The author makes no guarantees regarding the stability, compatibility, or safety of this library. Any device failures, data anomalies, financial losses, or other consequences arising from the use of this library are solely the responsibility of the user.

## Overview

A .NET port of the FANUC robot PC Interface, implemented based on the SNPX protocol. **No FRRJIf.dll or official dependencies required**.

Supports the following FANUC controllers (requires option **R651** or **R650+R553**):

- **R-J3iB**: 7D80/45+, 7D81/09+, 7D82/01+
- **R-J3iB Mate**: 7D91/01+
- **R-30iA / R-30iA Mate**: All versions
- **R-30iB / R-30iB Mate**: All versions
- **R-30iB Plus / Mate Plus / Compact Plus / Mini Plus**: All versions

**Option details:**
- **R651 (FRL Params)**: Built-in SNPX support, no additional option required
- **R650 (FRA Params)**: Requires **R553 (HMI Device SNPX)** to be selected together

## Installation

- **Target Framework**: .NET Standard 2.0 (compatible with .NET Framework 4.6.1+ / .NET Core 2.0+ / .NET 5+)
- **Dependencies**: No third-party libraries required. `Microsoft.Extensions.DependencyInjection` is needed only if using dependency injection.

## Quick Start

### Import Namespace

```csharp
using RJi.FanucRobot.Interface
```

### Configure RobotConnectionConfig

```csharp
// Default configuration (IP=127.0.0.1:60008)
var config = RobotConnectionConfig.Default;

// Custom configuration
var config = new RobotConnectionConfig
{
    IpAddress = "192.168.1.100",  // Robot IP
    Port = 60008,                 // SNPX port (default 60008)
    ConnectionTimeoutMs = 10000,  // Connection timeout (ms)
    ReadWriteTimeoutMs = 10000,   // Read/write timeout (ms)
    ClientId = 1024               // Client identifier (used on first connection, default 1024, usually no need to change)
};
```

### Create Client and Connect

```csharp
// Option 1: Default configuration (IP=127.0.0.1:60008)
using var robot = new FanucRobotClient();

// Option 2: Custom configuration
using var robot = new FanucRobotClient(config);

// Connect using the configuration passed to the constructor
bool connected = robot.Connect();              // Uses IpAddress + Port from config
bool connected = await robot.ConnectAsync();

// Or temporarily specify IP and port (overrides config)
bool connected = robot.Connect("192.168.1.100");
bool connected = await robot.ConnectAsync("192.168.1.100", port: 60008);
```

### Disconnect

```csharp
robot.Disconnect();
```

## Interface & Extensibility
This library exposes a single core interface **IFanucRobotClient** 
that enables full **dependency injection support** and unlimited extensibility.

By using this interface, you can easily:
- Register your robot client as a service with any DI container
- Create **unit tests** by mocking the interface
- Build **wrapper / decorator** patterns (logging, error handling, retry)
- Implement **observer / monitoring services** (background polling, data change notification)
- Push real-time data to subscribers (e.g., JSON messages, UI updates, IoT messages)
- Design clean, maintainable, and testable industrial applications

The interface keeps your core library clean 
while allowing YOU to design the upper-layer logic 
that best fits your application.

```csharp
services.AddSingleton<IFanucRobotClient>(new FanucRobotClient(config));
```

## API Overview

All APIs provide both synchronous and asynchronous versions. Async methods end with the `Async` suffix.

---

### Digital Signals

| Property    | Robot Signal       | Description             |
| ----------- | ------------------ | ----------------------- |
| `robot.DI`  | SDI                | Standard digital input  |
| `robot.DO`  | SDO                | Standard digital output |
| `robot.RI`  | RDI                | Robot input             |
| `robot.RO`  | RDO                | Robot output            |
| `robot.UI`  | User panel input   | UI                      |
| `robot.UO`  | User panel output  | UO                      |
| `robot.SI`  | System input       | SI                      |
| `robot.SO`  | System output      | SO                      |
| `robot.WI`  | Weld input         | WI                      |
| `robot.WO`  | Weld output        | WO                      |
| `robot.WSI` | Weld system input  | WSI                     |
| `robot.WSO` | Weld system output | WSO                     |

```csharp
// Read DI[1] (standard digital input channel 1)
bool val = robot.DI.ReadSingle(1);
bool val = await robot.DI.ReadSingleAsync(1);

// Batch read DO[1]~DO[10]
bool[] vals = robot.DO.Read(1, 10);
bool[] vals = await robot.DO.ReadAsync(1, 10);

// Write DO[1]=ON (set DO[1])
robot.DO.WriteSingle(1, true);
await robot.DO.WriteSingleAsync(1, true);

// Batch write DO[1]~DO[3] (DO[1]=ON, DO[2]=OFF, DO[3]=ON)
robot.DO.Write(1, new[] { true, false, true });
await robot.DO.WriteAsync(1, new[] { true, false, true });
```

> RI/RO/UI/UO/SI/SO/WI/WO/WSI/WSO provide the exact same ReadSingle/Read/WriteSingle/Write and Async methods.

---

### Group Signals

| Property   | Robot Signal | Description              |
| ---------- | ------------ | ------------------------ |
| `robot.GI` | GI           | Group input (int value)  |
| `robot.GO` | GO           | Group output (int value) |

```csharp
// Read GI[1] (group input channel 1)
int val = robot.GI.ReadSingle(1);
int val = await robot.GI.ReadSingleAsync(1);

// Batch read GI[1]~GI[5]
int[] vals = robot.GI.Read(1, 5);
int[] vals = await robot.GI.ReadAsync(1, 5);

// Write GO[1]=100 (group output channel 1 set to 100)
robot.GO.WriteSingle(1, 100);
await robot.GO.WriteSingleAsync(1, 100);

// Batch write GO[1]~GO[3] (GO[1]=100, GO[2]=200, GO[3]=300)
robot.GO.Write(1, new[] { 100, 200, 300 });
await robot.GO.WriteAsync(1, new[] { 100, 200, 300 });
```

---

### Analog Signals

| Property   | Robot Signal | Description         |
| ---------- | ------------ | ------------------- |
| `robot.AI` | AI           | Analog input (int)  |
| `robot.AO` | AO           | Analog output (int) |

```csharp
// Read AI[1] (analog input channel 1)
int ai = robot.AI.ReadSingle(1);
int ai = await robot.AI.ReadSingleAsync(1);

// Batch read AI[1]~AI[5]
int[] ais = robot.AI.Read(1, 5);
int[] ais = await robot.AI.ReadAsync(1, 5);

// Write AO[1]=500 (analog output channel 1 set to 500)
robot.AO.WriteSingle(1, 500);
await robot.AO.WriteSingleAsync(1, 500);

// Batch write AO[1]~AO[3]
robot.AO.Write(1, new[] { 100, 200, 300 });
await robot.AO.WriteAsync(1, new[] { 100, 200, 300 });
```

---

### PMC Signals

| Property    | Description                                             |
| ----------- | ------------------------------------------------------- |
| `robot.Pmc` | PMC signal access (R relay, K keep relay, D data table) |

```csharp
// Read R[1] (PMC internal relay, bit access)
bool r = robot.Pmc.ReadRelay(1);
bool r = await robot.Pmc.ReadRelayAsync(1);

// Write R[1]=ON
robot.Pmc.WriteRelay(1, true);
await robot.Pmc.WriteRelayAsync(1, true);

// Read K[1] (PMC keep relay)
bool k = robot.Pmc.ReadKeep(1);
bool k = await robot.Pmc.ReadKeepAsync(1);

// Write K[1]=ON
robot.Pmc.WriteKeep(1, true);
await robot.Pmc.WriteKeepAsync(1, true);

// Read D[1] (PMC data table, word access)
short d = robot.Pmc.ReadData(1);
short d = await robot.Pmc.ReadDataAsync(1);

// Write D[1]=12345
robot.Pmc.WriteData(1, 12345);
await robot.Pmc.WriteDataAsync(1, 12345);
```

---

### Numeric Registers R[i]

```csharp
// Read R[1] (float value)
float val = robot.NumReg.Read(1);
float val = await robot.NumReg.ReadAsync(1);

// Write R[1]=3.14
robot.NumReg.Write(1, 3.14f);
await robot.NumReg.WriteAsync(1, 3.14f);

// Batch read R[1]~R[10]
float[] vals = robot.NumReg.ReadBatch(1, 10);
float[] vals = await robot.NumReg.ReadBatchAsync(1, 10);

// Batch write R[1]~R[3]
robot.NumReg.WriteBatch(1, new float[] { 1.1f, 2.2f, 3.3f });
await robot.NumReg.WriteBatchAsync(1, new float[] { 1.1f, 2.2f, 3.3f });
```

> R[i] are inherently contiguous. `ReadBatchAsync` internally performs a single batch mapping + batch READ for optimal performance. No pre-registration required.

---

### Position Registers PR[i]

```csharp
// Read PR[1] (Group 1, includes joint coordinates, Cartesian coordinates, config word, tool/user frames)
PositionInfo pos = robot.PosReg.Read(1, group: 1);
PositionInfo pos = await robot.PosReg.ReadAsync(1, group: 1);

// Read PR[G2:1] (Group 2, multiple motion groups)
// ⚠ If the robot is not configured with group 2, no exception is thrown, but zero-filled data is returned
PositionInfo pos = robot.PosReg.Read(1, group: 2);
PositionInfo pos = await robot.PosReg.ReadAsync(1, group: 2);

// Batch read PR[1]~PR[5] (Group 1)
PositionInfo[] posList = robot.PosReg.ReadBatch(1, 5);
PositionInfo[] posList = await robot.PosReg.ReadBatchAsync(1, 5);

// Write PR[1] with joint coordinates (UF=1, UT=1)
var joint = new JointPosition { J1 = 0, J2 = -90, J3 = 0, J4 = 0, J5 = 0, J6 = 0 };
robot.PosReg.WriteJoint(1, joint, uf: 1, ut: 1);
await robot.PosReg.WriteJointAsync(1, joint, uf: 1, ut: 1);

// Write PR[2] with Cartesian coordinates (UF=1, UT=1)
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

// Batch write PR[1]~PR[3] with joint coordinates (simplified: same group, same UF/UT)
var joints = new[]
{
    new JointPosition { J1 = 0, J2 = -90, J3 = 0, J4 = 0, J5 = 0, J6 = 0 },
    new JointPosition { J1 = 30, J2 = -60, J3 = 30, J4 = 0, J5 = 90, J6 = 0 },
    new JointPosition { J1 = 60, J2 = -30, J3 = 60, J4 = 45, J5 = 45, J6 = 30 }
};
robot.PosReg.WriteJointBatch(1, joints, uf: 1, ut: 1);
await robot.PosReg.WriteJointBatchAsync(1, joints, uf: 1, ut: 1);

// Batch write PR[1]~PR[3] with joint coordinates (full: per-PR independent specification)
var ufArray = new short[] { 1, 2, 1 };   // Using UF1, UF2, UF1 respectively
var utArray = new short[] { 1, 1, 2 };   // Using UT1, UT1, UT2 respectively
var groups = new int[] { 1, 1, 2 };      // PR[1], PR[2] in group 1, PR[3] in group 2
robot.PosReg.WriteJointBatch(1, joints, ufArray, utArray, groups);
await robot.PosReg.WriteJointBatchAsync(1, joints, ufArray, utArray, groups);

// Batch write PR[10]~PR[12] with Cartesian coordinates (simplified: same group, same UF/UT)
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

// Batch write PR[10]~PR[12] with Cartesian coordinates (full: per-PR independent specification)
robot.PosReg.WriteCartesianBatch(10, carts, configs, ufArray, utArray, groups);
await robot.PosReg.WriteCartesianBatchAsync(10, carts, configs, ufArray, utArray, groups);
```

**`PositionConfig` Property Descriptions:**

| Property    | Type                        | Enum Values            | Robot Config String Meaning                                    |
| ----------- | --------------------------- | ---------------------- | -------------------------------------------------------------- |
| `NonFFlip`  | `PositionConfig.FlipState`  | `NonFlip=0` / `Flip=1` | F=No flip, FL=Flip                                             |
| `LeftRight` | `PositionConfig.HandConfig` | `Left=0` / `Right=1`   | L=Left-hand, R=Right-hand (**not displayed on teach pendant**) |
| `DownUp`    | `PositionConfig.ArmConfig`  | `Down=0` / `Up=1`      | D=Elbow down, U=Elbow up                                       |
| `BackTurn`  | `PositionConfig.TurnConfig` | `Back=0` / `Turn=1`    | B=Back, T=Turn (front)                                         |
| `Turn1`     | `short`                     | 0~7                    | J1 rotation count                                              |
| `Turn2`     | `short`                     | 0~7                    | J2 rotation count                                              |
| `Turn3`     | `short`                     | 0~7                    | J3 rotation count                                              |

> FANUC config string format is `[F, L, D, B, 0, 0, 0]`, mapping to `NonFlip, Left, Down, Back, Turn1=0, Turn2=0, Turn3=0`.

---

### String Registers SR[i]

```csharp
// Read SR[1]
string str = robot.StrReg.Read(1);
string str = await robot.StrReg.ReadAsync(1);

// Write SR[1]="HELLO"
robot.StrReg.Write(1, "HELLO");
await robot.StrReg.WriteAsync(1, "HELLO");

// Batch read SR[1]~SR[10]
string[] strs = robot.StrReg.ReadBatch(1, 10);
string[] strs = await robot.StrReg.ReadBatchAsync(1, 10);

// Batch write SR[1]~SR[3]
robot.StrReg.WriteBatch(1, new[] { "HELLO", "WORLD", "TEST" });
await robot.StrReg.WriteBatchAsync(1, new[] { "HELLO", "WORLD", "TEST" });
```

---

### Flag Registers F[i]

```csharp
// Read F[1] (0/1)
bool flag = robot.Flag.Read(1);
bool flag = await robot.Flag.ReadAsync(1);

// Write F[1]=ON
robot.Flag.Write(1, true);
await robot.Flag.WriteAsync(1, true);

// Batch read F[1]~F[20]
bool[] flags = robot.Flag.ReadBatch(1, 20);
bool[] flags = await robot.Flag.ReadBatchAsync(1, 20);

// Batch write F[1]~F[5] (F[1]=ON, F[2]=OFF, F[3]=ON, F[4]=OFF, F[5]=ON)
robot.Flag.WriteBatch(1, new[] { true, false, true, false, true });
await robot.Flag.WriteBatchAsync(1, new[] { true, false, true, false, true });
```

---

### System Variables

```csharp
// Supported types:
//   INT (4-byte integer), REAL (float), BOOL (boolean), STRING, POS (position)

// ⚠ If the variable name does not exist or is misspelled, no exception is thrown; zero/empty data is returned
// Verify the variable name beforehand and validate the return value for reasonableness

// Read $MOR_GRP[1].$ANGTOL[1] (integer)
int intVal = robot.SystemVariables.ReadInt("$MOR_GRP[1].$ANGTOL[1]");
int intVal = await robot.SystemVariables.ReadIntAsync("$MOR_GRP[1].$ANGTOL[1]");

// Read $SCRN_OP[1].$MAX_RPM (float)
float realVal = robot.SystemVariables.ReadFloat("$SCRN_OP[1].$MAX_RPM");
float realVal = await robot.SystemVariables.ReadFloatAsync("$SCRN_OP[1].$MAX_RPM");

// Read $SS_ENB (boolean)
bool boolVal = robot.SystemVariables.ReadBool("$SS_ENB");
bool boolVal = await robot.SystemVariables.ReadBoolAsync("$SS_ENB");

// Read $SYSNAME (string)
string strVal = robot.SystemVariables.ReadString("$SYSNAME");
string strVal = await robot.SystemVariables.ReadStringAsync("$SYSNAME");

// Read $MNUFRAME[1] (position variable)
PositionInfo posVal = robot.SystemVariables.ReadPosition("$MNUFRAME[1]");
PositionInfo posVal = await robot.SystemVariables.ReadPositionAsync("$MNUFRAME[1]");

// Write $MOR_GRP[1].$ANGTOL[1]=5
robot.SystemVariables.WriteInt("$MOR_GRP[1].$ANGTOL[1]", 5);
await robot.SystemVariables.WriteIntAsync("$MOR_GRP[1].$ANGTOL[1]", 5);

// Batch variable group (pre-register → bulk read, optimal performance)
var group = robot.SystemVariables.CreateVariableGroup(new List<(string, VariableType)>
{
    ("$MOR_GRP[1].$ANGTOL[1]", VariableType.INT),
    ("$SCRN_OP[1].$MAX_RPM",   VariableType.REAL),
    ("$SYSNAME",                VariableType.STRING)
});

// Full read (1 READ interaction, ideal for loop monitoring)
object[] batch = group.Read();
object[] batch = await group.ReadAsync();

// Full write
group.Write(new object[] { 100, 2000.0f, "Hello" });
await group.WriteAsync(new object[] { 100, 2000.0f, "Hello" });
```

---

### Current Position

```csharp
// World Cartesian coordinates (Group 1)
CartesianPosition cart = robot.Position.ReadWorldPosition(group: 1);
CartesianPosition cart = await robot.Position.ReadWorldPositionAsync(group: 1);

// Joint coordinates (Group 1)
JointPosition joint = robot.Position.ReadJointPosition(group: 1);
JointPosition joint = await robot.Position.ReadJointPositionAsync(group: 1);

// Full position under specified UF (includes joint + Cartesian + config)
PositionInfo pos = robot.Position.ReadUserPosition(ufNumber: 1, group: 1);
PositionInfo pos = await robot.Position.ReadUserPositionAsync(ufNumber: 1, group: 1);
```

---

### Task Manager

```csharp
// PRG[1] monitor task 1 status
TaskInfo task = robot.Task.Read(index: 1);
TaskInfo task = await robot.Task.ReadAsync(index: 1);

// Ignore Karel tasks (PRG[K1])
TaskInfo task = await robot.Task.ReadAsync(index: 1, type: TaskType.IgnoreKarel);
```

**TaskType Enum:**

| Enum Value         | SETASG Format | Description                       |
| ------------------ | ------------- | --------------------------------- |
| `Normal`           | PRG[i]        | Standard task monitoring          |
| `IgnoreKarel`      | PRG[K{i}]     | Ignore Karel tasks                |
| `IgnoreMacro`      | PRG[M{i}]     | Ignore macro tasks                |
| `IgnoreMacroKarel` | PRG[MK{i}]    | Ignore both Karel and macro tasks |

**TaskInfo Properties:**

| Property         | Type   | Description                                                |
| ---------------- | ------ | ---------------------------------------------------------- |
| `ProgName`       | string | Name of the currently running program                      |
| `LineNumber`     | int    | Current line number                                        |
| `State`          | int    | Task status code (1=Running, 2=Paused, 3=Stopped, 4=Error) |
| `StateText`      | string | Status description (RUN / PAUSE / STOP / ERROR)            |
| `ParentProgName` | string | Parent program name (valid when called as a subprogram)    |

---

### Alarm Manager

```csharp
// E1 alarm history (List mode, Full format, top 10 entries)
AlarmItem[] alarms = robot.Alarm.Read(count: 10);
AlarmItem[] alarms = await robot.Alarm.ReadAsync(count: 10);

// Current alarms (Current mode, Short format)
AlarmItem[] current = await robot.Alarm.ReadAsync(
    count: 10,
    type: AlarmType.Current,
    mode: AlarmMessageMode.Short
);

// Clear alarms (type=0 clears current alarms)
robot.ClearAlarm(type: 0);
await robot.ClearAlarmAsync(type: 0);
```

**AlarmType Enum:**

| Enum Value | SETASG Format | Description                  |
| ---------- | ------------- | ---------------------------- |
| `List`     | ALM[E1]       | Alarm history list (default) |
| `Current`  | ALM[1]        | Current alarms               |
| `Password` | ALM[P1]       | Password alarms              |

**AlarmMessageMode Enum:**

| Enum Value   | Words/Entry | Description                                              |
| ------------ | ----------- | -------------------------------------------------------- |
| `Full` (0)   | 100         | Full: AlarmMessage + CauseAlarmMessage + SeverityMessage |
| `Short` (1)  | 51          | Basic fields only + AlarmMessage                         |
| `Medium` (2) | 91          | Basic fields + AlarmMessage + CauseAlarmMessage          |

**AlarmItem Properties:**

| Property            | Type      | Description                                                          |
| ------------------- | --------- | -------------------------------------------------------------------- |
| `AlarmId`           | short     | Alarm ID (accumulated count for multiple alarms at same location)    |
| `AlarmNumber`       | short     | Alarm number (alarm code defined by the controller)                  |
| `CauseAlarmId`      | short     | Cause alarm ID (0=none)                                              |
| `CauseAlarmNumber`  | short     | Cause alarm number                                                   |
| `Severity`          | short     | Severity level (raw code, refer to teach pendant for interpretation) |
| `AlarmMessage`      | string    | Alarm message text (empty in Short mode)                             |
| `CauseAlarmMessage` | string    | Cause alarm message (Medium/Full mode)                               |
| `SeverityMessage`   | string    | Severity level description (Full mode only)                          |
| `Timestamp`         | DateTime? | Alarm time (merged year/month/day/hour/minute/second)                |

---

### Comment Manager

```csharp
// Read via enum SI[C1] (system input channel 1 comment)
string comment = robot.Comment.Read(CommentType.SI, index: 1);
string comment = await robot.Comment.ReadAsync(CommentType.SI, index: 1);

// Read via string prefix
string comment = robot.Comment.Read("DI", 1);
string comment = await robot.Comment.ReadAsync("SR", 1);

// Write comment via enum DO[C1] (digital output channel 1 comment)
robot.Comment.Write(CommentType.DO, index: 1, value: "Gripper Station");
await robot.Comment.WriteAsync(CommentType.DO, index: 1, value: "Gripper Station");

// Write comment via string prefix
robot.Comment.Write("R", 5, "Temperature Threshold");
await robot.Comment.WriteAsync("SR", 1, "Product Name");
```

**CommentType Enum (19 types total):**

Signal comments: `DI` `DO` `RI` `RO` `UI` `UO` `SI` `SO` `GI` `GO` `AI` `AO` `WI` `WO` `WSI` `WSO`

Register/POS comments: `R` (numeric register) `SR` (string register) `PR` (position register)

---

### Connection Status & Error Handling

```csharp
// Check connection status
bool isConnected = robot.IsConnected;

// You can read the original connection configuration at any time
string ip = robot.Config.IpAddress;
int port = robot.Config.Port;

// Error handling via RobotException
try
{
    float val = await robot.NumReg.ReadAsync(1);
}
catch (RobotException ex)
{
    Console.WriteLine($"[{ex.ErrorCode}] {ex.Message}");
    // Connection errors require destroying the client and recreating it
    robot.Dispose();
}
```

| Error Code                                                     | Description                |
| -------------------------------------------------------------- | -------------------------- |
| `NotConnected`                                                 | Not connected to the robot |
| `ConnectionFailed` / `ConnectionTimeout` / `ConnectionRefused` | Connection failure         |
| `ProtocolError` / `InvalidResponse`                            | Protocol/response error    |
| `ReadError` / `WriteError`                                     | Read/write error           |
| `InvalidAddress` / `InvalidData`                               | Parameter error            |

## Comparison with Official Library API

| Feature                    | Official Library (FRRJIf.dll)                                   | This Library                                                    |
| -------------------------- | --------------------------------------------------------------- | --------------------------------------------------------------- |
| **Dependencies**           | Requires FRRJIf.dll installation, Windows only                  | Pure .NET Standard 2.0, cross-platform                          |
| **Connection**             | `Core.Connect(hostName)`                                        | `robot.Connect(ip, port)` / parameterless overload              |
| **Disconnection**          | `Core.Disconnect()`                                             | `robot.Disconnect()`                                            |
| **Digital Signals**        | `ReadSdo/Sdi/Rdo/Rdi/So/Si/Uo/Ui` passing `Array` + `Count`     | `robot.DI.ReadSingle/Read` directly returns `bool`/`bool[]`     |
| **Group Signals**          | `ReadGo/Gi/WriteGo/Gi` passing `Array`                          | `robot.GI.Read/Write` directly returns `int[]`                  |
| **Analog Signals**         | Indirect access via GI/GO index +1000                           | `robot.AI/AO.ReadSingle/WriteSingle` direct read/write          |
| **PMC Signals**            | `ReadSdo(Index≥11001)` + `ReadPmcr2`                            | `robot.Pmc.ReadRelay/ReadKeep/ReadData` categorized access      |
| **Numeric Registers**      | `AddNumReg → GetValue/SetValues` (requires DataTable pre-reg.)  | `robot.NumReg.Read/Write/ReadBatch` ready-to-use                |
| **Position Registers**     | `AddPosReg` or `AddPosRegXyzwpr` + `Refresh`                    | `robot.PosReg.Read/WriteJoint/WriteCartesian`                   |
| **String Registers**       | Indirect via `AddSysVar(STRREG)`                                | `robot.StrReg.Read/Write/ReadBatch`                             |
| **Flag Registers**         | `AddFlag` + `Refresh`                                           | `robot.Flag.Read/Write/ReadBatch`                               |
| **System Variables**       | `AddSysVar/AddSysVarPos` + `Refresh` (one-by-one registration)  | `robot.SystemVariables.ReadXxx/WriteXxx` direct read/write      |
| **System Variables Batch** | One-by-one AddSysVar, limited by DataTable capacity             | `CreateVariableGroup` one-time registration, batch read/write   |
| **Current Position**       | `AddCurPosUF` + `Refresh`                                       | `robot.Position.ReadWorldPosition/ReadJointPosition`            |
| **Task Monitoring**        | `AddTask` + `Refresh`                                           | `robot.Task.Read` supports multiple TaskTypes                   |
| **Alarm Management**       | `AddAlarm` + `GetValue` (requires type and count specification) | `robot.Alarm.Read` supports AlarmType + AlarmMessageMode        |
| **Comment Read/Write**     | Indirect via `AddSysVar(XX_COMMENT)`                            | `robot.Comment.Read/Write` supports CommentType enum or string prefix |
| **Clear Alarm**            | `ClearAlarm(type)`                                              | `robot.ClearAlarm/ClearAlarmAsync`                              |
| **Capacity Limit**         | DataTable fixed size, must create new DataTable if exceeded     | Three-level cache auto-management, no theoretical upper limit   |
| **Data Management**        | Manual: AddXXX → Refresh → GetValue                             | Auto SETASG + caching, ready-to-use                             |
| **Sync Version**           | Sync only                                                       | Sync + Async (`Async` suffix)                                   |
| **Dependency Injection**   | Not supported                                                   | Supports MSDI registration                                      |
