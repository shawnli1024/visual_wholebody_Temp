先说结论：部署流程分四步 —— 验证 & 导出、打包观测预处理、在实机上运行推理并发布命令、上电前的安全检查。下面给出精简可执行步骤与注意事项。

1-在仿真中验证并导出策略（生成 TorchScript）
    确认在本地能用 play.py 正常加载并运行你要部署的 checkpoint（如 model_38000.pt）。
    导出为 TorchScript:可在 play.py 中启用 EXPORT_POLICY 或运行一个小脚本加载训练好的 runner 后用 torch.jit.trace/torch.jit.script 导出 actor（保存到 logs/.../exported/policies/）。
    也可用 SAVE_ACTOR_HIST_ENCODER 导出 actor 权重（便于后续打包）。
Exported policy as jit script to: 
    /home/lisy/Work/MySpace/005_WBC/visual_wholebody/low-level/logs/b1z1_v2/exported/policies/policy_1.pt

2-把观测预处理与动作后处理一并打包
部署时要保证仿真/训练中的观测归一化（mean/std）、history 编码器等处理一致。最佳做法是把 preprocessing（归一化、拼接、历史编码）和 actor 包成一个 TorchScript 模块，这样实机上只需给原始传感器数据，模型直接输出控制量。
若无法打包，需在机器人端复现训练时的归一化参数并严格按同样顺序拼接 obs。

3-在机器人端选择推理方式 & 运行控制环
推理方式：Python (torch.jit.load) 或 C++ (libtorch)。推荐用 Python 快速验证，生产上用 libtorch/C++ 更稳定、延迟更小。
控制频率：按低层控制频率运行（训练时的低层频率，常见 100–200 Hz），保证实时性与固定周期。
输入/输出映射：确认 actor 输出类型（关节位置/速度/期望加速度/关节力矩），以及 DOF 顺序与机器人控制器一致。实现动作限幅、平滑（低通）和安全边界（最大力矩/速度）。
通信：用你机器人的控制框架（ROS/ROS2、CAN、UDP 等）把动作发送给底层控制器；注意时间戳与丢包处理。

4-上机前的安全与测试流程
离线 sanity check：用已导出的模型在本地用录制的传感器数据回放，确认输出合理且无 NaN/溢出。
软限制与 watchdog：在机器人端实现力/位置安全阈值和超时 watchdog，一旦异常立即切换到安全模式（零力或停止）。
小步上电测试：先把机器人固定在台架，低频（如 1–5 Hz）测试动作输出，再逐步恢复目标控制频率；观察电流、温度和位置。
最后在开放场景下逐步增加行为复杂度并监控日志。

5-常见坑与提示（必读）

观测顺序和维度必须完全一致；训练时若用了 privileged info，部署时不可使用这些未能获得的项。
归一化参数（mean/std/clip）很关键，缺失会导致策略输出异常。
CUDA 与 libtorch 版本兼容性：若在机器人上用 GPU 推理，libtorch/torchscript 编译时要与目标设备的 CUDA/driver 匹配，版本不一致会导致加载失败。
实时性：Python + GIL 可能无法稳定维持高频控制，生产上优先使用 C++/libtorch。
若策略输出的是高频 torque，确保底层电机驱动支持并安全限幅；若是低频目标位置，可在实机做独立的低层跟踪控制器。

6-快速命令示例（导出参考）
在本地把 actor 导出为 TorchScript（示例思路）：
加载 runner -> 取 actor 模块 -> 用示例 obs 调用 torch.jit.trace -> 保存为 .pt
在机器人上加载（Python）：
model = torch.jit.load("actor_traced.pt", map_location='cpu') -> 在控制回路里 model(input_tensor) -> 将输出映射到机器人命令并发送