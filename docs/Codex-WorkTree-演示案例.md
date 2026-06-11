# Codex WorkTree 演示案例：基于 WPFDemo 的并行改动演示

## 1. 演示目标

用当前仓库 `D:\WorkCode\WPFDemo` 做一个现场演示，说明 Codex WorkTree 的三个核心价值：

1. 隔离：Codex 在独立 worktree 中修改代码，不打断当前本地 checkout。
2. 并行：本地可以继续开发，Codex 可以在另一个工作区完成小任务。
3. 可审查：最终通过 diff 和 build 结果决定是否接收改动。

演示成功标准：

1. 能说明当前项目的真实结构和 baseline 状态。
2. 能让 Codex 在 WorkTree 中完成一个小功能。
3. 能通过 `dotnet build WPFDemo.sln` 验证改动。
4. 能展示改动只集中在必要文件中。

## 2. 当前项目背景

当前仓库是一个 WPF Demo 项目：

- 解决方案：`D:\WorkCode\WPFDemo\WPFDemo.sln`
- 主窗口界面：`D:\WorkCode\WPFDemo\WPFDemo\MainWindow.xaml`
- 主窗口逻辑：`D:\WorkCode\WPFDemo\WPFDemo\MainWindow.xaml.cs`

界面里已有两个按钮：

- 第一个按钮显示文字 `Button`，点击后弹出 `Hello`。
- 第二个按钮显示文字 `WorkTree按钮`，点击后弹出 `我是WorkTree按钮`。
- 右侧有一个标题文本，当前显示 `Title`。

这个项目很适合做 WorkTree 演示，因为功能小、文件少、验证方式清晰。

## 3. 演示场景

假设我们正在本地继续开发 WPF 项目，但希望让 Codex 在不影响当前工作区的情况下，帮我们改一个小功能：

> 点击 `WorkTree按钮` 时，不再弹出消息框，而是把右侧标题文本改成 `WorkTree 演示已完成`。

这个任务的边界很清楚：

- 只涉及主窗口 UI 和按钮事件。
- 不需要新增复杂架构。
- 不需要引入依赖。
- 验证方式是编译通过，并检查 diff。

## 4. 现场演示流程

### 第一步：展示 baseline

先在本地终端展示当前仓库状态：

```powershell
cd D:\WorkCode\WPFDemo
git status --short --branch
dotnet build .\WPFDemo.sln
```

讲解词：

> 当前项目在 `main` 分支，工作区是干净的。我们先确认 baseline 可以正常编译，这样后面 WorkTree 中的改动是否引入问题就能独立判断。

预期结果：

- `git status` 显示工作区干净。
- `dotnet build` 显示 `0 个警告`、`0 个错误`。

### 第二步：在 Codex 中创建 WorkTree 任务

在 Codex app 中新建一个 thread，选择当前项目 `D:\WorkCode\WPFDemo`，环境选择 `Worktree`，起点选择当前分支 `main`。

发给 Codex 的 Prompt：

```text
请在当前 WPF 项目中做一个小改动，用于演示 WorkTree 功能。

目标：
点击“WorkTree按钮”时，不再弹出 MessageBox，而是把右侧标题文本从 Title 更新为“WorkTree 演示已完成”。

约束：
- 只改必要文件。
- 保留第一个 Button 的现有行为。
- 不引入新依赖。
- 不做额外重构。

验证：
完成后运行 dotnet build WPFDemo.sln，并告诉我修改了哪些文件。
```

讲解词：

> 这个任务会在独立 WorkTree 中执行。Codex 可以读代码、改代码、运行 build，但这些中间改动不会直接混进我当前的本地 checkout。

### 第三步：展示本地 checkout 没有被打扰

在 Codex WorkTree 任务运行时，回到本地终端：

```powershell
cd D:\WorkCode\WPFDemo
git status --short --branch
```

讲解词：

> 注意这里还是我们原来的本地 checkout。WorkTree 里的文件改动不会直接出现在这里，所以我可以继续做手上的事情，不需要先 stash，也不需要担心 Codex 的中间状态污染当前工作区。

如果想把隔离效果讲得更明显，可以在本地临时做一个不提交的小改动，例如只改窗口标题，然后再次展示 WorkTree 线程的 diff 和本地 diff 是分开的。演示结束后记得还原这个临时改动。

### 第四步：查看 WorkTree 产出的 diff

Codex 完成后，在 Codex app 里查看 diff。

预期改动大致应该是：

1. `MainWindow.xaml`
   - 给右侧 `TextBlock` 增加一个 `x:Name`，例如 `TitleTextBlock`。

2. `MainWindow.xaml.cs`
   - 修改 `WorkTreeButton_Click`。
   - 将原来的 `MessageBox.Show("我是WorkTree按钮");` 改为更新标题文本：

```csharp
TitleTextBlock.Text = "WorkTree 演示已完成";
```

讲解词：

> 这里最重要的不是 Codex 写了多少代码，而是我们能看到它到底改了什么。这个案例里，合理的结果应该只改两个文件，而且每一行都能对应到需求。

### 第五步：验证结果

在 WorkTree 线程中确认 Codex 已经运行：

```powershell
dotnet build WPFDemo.sln
```

也可以在接收改动后，本地再次运行：

```powershell
cd D:\WorkCode\WPFDemo
dotnet build .\WPFDemo.sln
```

讲解词：

> WorkTree 不是跳过验证，而是把验证前移到独立工作区里。我们先让 Codex 在 WorkTree 里跑 build，再决定要不要把这份改动接回本地。

### 第六步：接收或继续迭代

如果 diff 和 build 都符合预期，可以选择：

- 让 Codex 继续补充说明或微调。
- 使用 Handoff 把这个 thread 的工作切回 Local。
- 在 Codex app 中 stage、commit、push。
- 或者手动把改动合并到当前工作分支。

讲解词：

> WorkTree 的价值在这里体现得最明显：它不是替你跳过工程流程，而是让 AI 的改动以一个可审查、可验证、可回退的 Git 结果出现。

## 5. 推荐演示节奏

总时长建议控制在 6 到 8 分钟：

1. 1 分钟：介绍项目结构和现有按钮。
2. 1 分钟：说明为什么不用 Local，改用 WorkTree。
3. 2 分钟：创建 WorkTree 任务并发送 Prompt。
4. 1 分钟：展示本地 checkout 没被打扰。
5. 2 分钟：查看 diff 和 build 结果。
6. 1 分钟：总结 WorkTree 适合什么任务。

## 6. 讲解重点

可以反复强调这三句话：

1. WorkTree 给 Codex 一个独立施工区。
2. 本地开发和 AI 后台任务可以并行。
3. 最终是否接收改动，仍然通过 diff、build、review 决定。

这个案例里，Codex 做的是一个很小的 UI 行为调整，但它完整展示了真实工程协作流程：

- 有明确需求。
- 有独立工作区。
- 有最小改动。
- 有构建验证。
- 有人工审查。

## 7. 常见问答

问：WorkTree 是不是会自动合并到我当前代码里？

答：不是。WorkTree 的重点是隔离。它让 Codex 在独立工作区完成任务，最终仍然需要你审查 diff，再决定是否接收、提交或创建 PR。

问：什么时候适合用 WorkTree？

答：适合边界清晰、可以独立验证的任务，比如修一个小 bug、补测试、改一个 UI 行为、升级前做探索、生成文档或检查报告。

问：什么时候不一定要用 WorkTree？

答：如果任务需要你立刻在本地 IDE 里断点调试，或者强依赖当前未提交的本地状态，直接在 Local 里和 Codex 协作可能更顺手。

问：这个 WPF 项目的演示为什么选按钮行为？

答：因为它足够小，能避免演示被复杂业务逻辑打断；同时它又是真实代码改动，能展示 XAML、事件处理、diff 和 build 验证。

## 8. 收尾总结

这次演示可以这样收尾：

> 在这个 WPFDemo 项目里，我们让 Codex 在 WorkTree 中独立完成了一个按钮行为修改。本地 checkout 没被打扰，Codex 的改动可以单独审查，并且通过 build 验证。WorkTree 的核心价值就是把 AI 编程从“直接改我的工作区”，变成“在独立工作区完成任务，然后把结果交给我审查”。

