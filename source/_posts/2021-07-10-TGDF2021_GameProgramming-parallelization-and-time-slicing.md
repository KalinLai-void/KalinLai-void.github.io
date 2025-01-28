---
title: 【TGDF2021】遊戲程式優化技巧：平行化與時間切割（Unity）
author: Karlun Lai
date: 2021-07-10 09:04:45
tags:
  - TGDF台北遊戲開發者論壇
  - TGDF2021
  - 遊戲程式
  - Unity
categories:
  - TGDF2021_台北遊戲開發者論壇
---
講者：周明倫
<!--more-->
[線上直播連結(代補)]()

講者介紹
---
- 頑皮狗遊戲工程師
- 工作專精：AI/遊戲機制
- 興趣研究：遊戲物理/圖學相關研究
- 曾參與作品：
  - Uncharted 4（秘境探險4）
  - Uncharted: The Lost Legacy（秘境探險：失落的遺產）
  - The Last of Us Part II（最後生還者2）
- 個人開發工具：
  - 體積特效工具 MudBun
  - 彈跳特效工具 Boing Kit
- [Blog](http://allenchou.net/)、[Twitter](https://twitter.com/theallenchou)

範例：曝光地圖 & 曝光迴避
---
此範例是**曝光地圖(Exposure Map)** 與 **曝光迴避(Exposure Avoidance)** 。

- 曝光地圖：由下圖的紅色與綠色網格所組成的資料，用以代表是否曝光於紅色正方體（哨兵）。<u>紅色即為哨兵的視野</u>。
- 曝光迴避：藍球會迴避哨兵的視野，當曝光於哨兵視野下時，則會去找附近未曝光的格子（綠色網格）進行迴避。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Exposure-Map-and-Exposure-Avoidance.gif)

這個範例是使用**射線投射(raycast)**的方式來偵測並更新曝光地圖，不一定要使用射線投射，在此僅因為範例才用射線投射。

## 單執行緒 (Single-Threading)

最直接的實作方式就是在主執行緒上進行曝光地圖的更新，在使用曝光迴避讓藍球來進行迴避，**本次講座將以「更新曝光地圖」的函式做為重點**。

曝光地圖內有一連串的射線投射，在下圖以黃色長方體為代表。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Single-Threading.png)

### 程式碼
```C#
void UpdateExposureMap()
{
  for (int iCell = 0; iCell < NumCells; ++iCell)
  {
    // 初始化射線投射
    Vector3 cellCenter = Grid[iCell];
    Vector3 vec = cellCenter - EyePos;
    Vector3 rayDir = vec.normalized;
    float rayLen = vec.magnitude;
 
    // 使用射線投射來計算是否暴露
    bool exposed = !Physics.Raycast(EyePos, rayDir, rayLen);
    ExposureMap[iCell] = exposed;
  }
}
```

### 性能分析器(Profiler)呈現結果
如下圖，可以看到主執行緒內的「更新曝光地圖」函式，裡面<u>塞滿一連串射線投射的計算（黃色區域）</u>。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Single-Threaded-Profiler.png)

而使用單執行緒方式實作時，CPU核心是停擺狀態，非常浪費，所以可以改為「多執行緒」的做法。

## 多執行緒 (Multi-Threading)

宏觀上步驟與單執行緒實作相同，差別只在「更新曝光地圖」的函式開工(Kick)拆成可以由不同執行緒來進行的小工作，而每個執行緒的射線投射數量就比單執行緒來的低，最後再收工(Gather)來更新曝光地圖。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Multi-Threading.png)

在此我們可以使用Unity的**工作系統(job system)** ，來實作多執行緒的運算。

### 程式碼
```C#
// 設置每個射線的起始值
struct RaycastSetupJob : IJobParallelFor
{
  public Vector3 EyePos;
 
  [ReadOnly]
  public NativeArray<Vector3> Grid;
 
  [WriteOnly]
  public NativeArray<RaycastCommand> Commands;
 
  // 每次被呼叫時，設置單一射線
  public void Execute(int index)
  {
    Vector3 cellCenter = Grid[index];
    Vector3 vec = cellCenter - EyePos;
    Commands[index] = 
      new RaycastCommand(EyePos, vec.normalized, vec.magnitude);
  }
}
 
// 讀取射線投射的結果，用以更新曝光地圖
struct RaycastGatherJob : IJobParallelFor
{
  [ReadOnly]
  public NativeArray<RaycastHit> Results;
 
  [WriteOnly]
  public NativeArray<bool> ExposureMap;
 
  // 每次被呼叫時，更新單一曝光地圖格子
  public void Execute(int index)
  {
    bool exposed = (Results[index].distance <= 0.0f);
    ExposureMap[index] = exposed;
  }
}
```

然後更新曝光地圖函式被改成以下：

```C#
void UpdateExposureMap()
{
  // allocate data shared across jobs
  var allocator = Allocator.TempJob;
  var commands = new NativeArray<RaycastCommand>(NumCells, allocator);
  var results = new NativeArray<RaycastHit>(NumCells, allocator);
 
  // create setup job
  var setupJob = new RaycastSetupJob();
  setupJob.EyePos = EyePos;
  setupJob.Grid = Grid;
  setupJob.Commands = commands;
 
  // create gather job
  var gatherJob = new RaycastGatherJob();
  gatherJob.Results = results;
  gatherJob.ExposureMap = ExposureMap;
 
  // kick
  // 以下3個function彼此相互依賴
  // 上面做完了才會往下做
  var hSetupJob = setupJob.Schedule(NumCells, JobBatchSize);
  var hRaycastJob = RaycastCommand.ScheduleBatch(commands, results, JobBatchSize, hSetupJob);
  var hGatherJob = gatherJob.Schedule(NumCells, JobBatchSize, hRaycastJob);
  JobHandle.ScheduleBatchedJobs();
 
  // gather
  hGatherJob.Complete();
 
  // dispose of job data
  commands.Dispose();
  results.Dispose();
}
```

### 性能分析器(Profiler)呈現結果
如下圖，可以看到所耗時間變成原本的10%，並且可以看到射線投射工作已被平行化運算，但<u>運算的時間範圍還是被包含在主更新函式的範圍內</u>。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Multi-Threaded-Profiler.png)

而在主更新函式內，我們最後是等待所有工作收工，所以在等待收工的期間，主執行緒還是空轉狀態。若可以負擔1幀的延遲，可以更進一步使用「延遲結果運算」來優化。

## 延遲蒐集運算結果 (Delayed Result Gathering)

此技巧又稱「異步」。
改為在「更新曝光地圖」函式的最後才開工，並且工作平行於「執行曝光迴避」，而在下一幀的「更新曝光地圖」才收工。

又因為這些工作與「執行曝光迴避」平行，因此有可能會造成**競爭條件(race condition)** 。而避免競爭條件的其中一個技巧是曝光地圖以「**雙重緩衝(double-buffered)** 」的方式儲存，射線投射相關工作的是對後緩衝(back buffer)進行更新，而執行曝光迴避時則是讀取前緩衝(front buffer)。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Delayed-Result-Gathering.png)

### 程式碼
大部分的東西是一樣的，差別在於：開工被改到了函式最結尾、收工被改到函式的開頭

```C#
void UpdateExposureMap()
{
  // gather
  hGatherJob.Complete();
 
  // double-buffering
  SwapExposureBackBuffer();
 
  // dispose of data allocated from last frame
  if (Commands.IsCreated)
    Commands.Dispose();
  if (Results.IsCreated)
    Results.Dispose();
 
  // allocate data shared across jobs
  var allocator = Allocator.TempJob;
  Commands = new NativeArray<RaycastCommand>(NumCells, allocator);
  Results = new NativeArray<RaycastHit>(NumCells, allocator);
 
  // create setup job
  var setupJob = new RaycastSetupJob();
  setupJob.EyePos = EyePos;
  setupJob.Grid = Grid;
  setupJob.Commands = Commands;
 
  // create gather job
  var gatherJob = new RaycastGatherJob();
  gatherJob.Results = Results;
  gatherJob.ExposureMap = ExposureMap;
 
  // kick
  var hSetupJob = setupJob.Schedule(NumCells, JobBatchSize);
  var hRaycastJob = RaycastCommand.ScheduleBatch(Commands, Results, JobBatchSize, hSetupJob);
  hGatherJob = gatherJob.Schedule(NumCells, JobBatchSize, hRaycastJob);
  JobHandle.ScheduleBatchedJobs();
}
```

### 性能分析器(Profiler)呈現結果
如下圖，可以看到主更新函式在主執行緒的長度幾乎降了8成，而在其他執行緒的時間範圍已經不和主更新函式重疊。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Delayed-Result-Gathering-Profiler.png)

## 時間切割(Time Slicing)
如果1幀的時間還是不夠計算，或者希望分配給射線投射的時間在少一點的話，可以使用「**時間切割**」。簡單來說就是<u>若能負擔數幀延遲的話，就將1幀的工作量拆成數幀來進行</u>。

下圖我們可以看到，每一幀並非所有射線投射都被更新，而是**每一幀只進行部分的射線投射，且只更新部分的曝光地圖** 。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Exposure-Map-and-Exposure-Avoidance-2.gif)

當整個批次的射線投射都被計算完後，更換曝光地圖的前後緩衝，因此會看到曝光地圖的更新稍微有些延遲，但又因曝光地圖是**非直接**的AI資料，因此<u>單看藍球的行為感覺不出曝光地圖更新的延遲</u>，但降低了每幀的計算量。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Time-Slicing.png)

### 程式碼
更改後的工作結構如下，大致上是一樣的，只是增加了時間切割的引數。

```C#
struct RaycastSetupJob : IJobParallelFor
{
  public Vector3 EyePos;
 
  [ReadOnly]
  public NativeArray<Vector3> Grid;
 
  [ReadOnly]
  public NativeArray<RaycastCommand> Commands;
 
  // 時間切割引數
  public int TimeSliceBaseIndex;
 
  public void Execute(int localIndex)
  {
    // 利用時間切割引數與區域引數相加取得全域引數
    int globalIndex = localIndex + TimeSliceBaseIndex;
    Vector3 cellCenter = Grid[globalIndex];
    Vector3 vec = cellCenter - EyePos;
    Commands[localIndex] = 
      new RaycastCommand(EyePos, vec.normalized, vec.magnitude);
  }
}
 
struct RaycastGatherJob : IJobParallelFor
{
  [ReadOnly]
  public NativeArray<RaycastHit> Results;

  // 原本此標籤為[WriteOnly]，但會報錯
  // 因為若區域引數與用來存取陣列的引數不同的話，Unity會認為有競爭條件的風險
  // 但若知道自己在做什麼，並且保證不會有競爭條件的話
  // 因此改為[NativeDisableParallelForRestriction]，就可以叫Unity不要那麼囉嗦
  [NativeDisableParallelForRestriction]
  public NativeArray<bool> ExposureMap;
 
  // 時間切割引數
  public int TimeSliceBaseIndex;
 
  public void Execute(int localIndex)
  {
    // 利用時間切割引數與區域引數相加取得全域引數
    int globalIndex = localIndex + TimeSliceBaseIndex;
    bool exposed = (Results[localIndex].distance <= 0.0f);

    // 全域引數用以更新曝光地圖的格子
    ExposureMap[globalIndex] = exposed;
  }
}
```

而更改後的更新函式如下，其實也差不多，差別在於「必須去追蹤時間切割引數的進度」

```C#
void UpdateExposureMap(int numRaysPerTimeSlice)
{
  // wait for jobs from last frame to complete
  hGatherJob.Complete();
 
  // trim excess ray count for the last time slice of batch
  int numExcessRays = TimeSliceBaseIndex + numRaysPerTimeSlice - NumCells;
  numRaysPerTimeSlice -= Mathf.Max(0, numExcessRays);
 
  // batch ended?
  if (TimeSliceBaseIndex < 0)
  {
    // double-buffering
    SwapExposureBackBuffer();
 
    // reset time slicing index
    TimeSliceBaseIndex = 0;
  }
 
  // dispose of job data allocated from last frame
  if (Commands.IsCreated)
    Commands.Dispose();
  if (Results.IsCreated)
    Results.Dispose();
 
  // allocate data shared across jobs
  var allocator = Allocator.TempJob;
  Commands = new NativeArray<RaycastCommand>(numRaysPerTimeSlice, allocator);
  Results = new NativeArray<RaycastHit>(numRaysPerTimeSlice, allocator);
   
  // create setup job
  var setupJob = new RaycastSetupJob();
  setupJob.EyePos = EyePos;
  setupJob.Grid = Grid;
  setupJob.Commands = Commands;
  setupJob.TimeSliceBaseIndex = TimeSliceBaseIndex;
   
  // create gather job
  var gatherJob = new RaycastGatherJob();
  gatherJob.Results = Results;
  gatherJob.ExposureMap = ExposureMap;
  gatherJob.TimeSliceBaseIndex = TimeSliceBaseIndex;
   
  // schedule setup job
  var hSetupJob = setupJob.Schedule(numRaysPerTimeSlice, JobBatchSize);
   
  // schedule raycast job
  // specify dependency on setup job
  var hRaycastJob = RaycastCommand.ScheduleBatch(Commands, Results, JobBatchSize, hSetupJob);
   
  // schedule gather job
  // specify dependency on raycast job
  hGatherJob = gatherJob.Schedule(numRaysPerTimeSlice, JobBatchSize, hRaycastJob);
 
  // advance time slice index
  TimeSliceBaseIndex += numRaysPerTimeSlice;
 
  // end of batch?
  if (TimeSliceBaseIndex >= NumCells)
  {
    // signal end of batch
    TimeSliceBaseIndex = -1;
  }
 
  // kick jobs
  JobHandle.ScheduleBatchedJobs();
}
```

## 效能數據

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Efficacy-Data-10k.png)

## 消除雙重緩衝
從下圖可看到，若要避免雙重緩衝，可以將「更新曝光地圖」函式拆成兩個「收工曝光地圖」與「開工曝光地圖」，需要讀取資料時不與工作重疊，因此也就不需要使用雙重緩衝。

![](/images/TGDF2021_GameProgramming-parallelization-and-time-slicing/Remove-Double-Buffered.png)

補充與參考資料
---
- 範例的Github repo：https://github.com/TheAllenChou/tgdf-2021
- 更詳細可以見講師的Blog文章：
  - [遊戲程式：延遲蒐集運算結果](http://allenchou.net/2021/05/delayed-result-gathering-chinese/)
  - [遊戲程式：時間切割](http://allenchou.net/2021/05/time-slicing-chinese/)