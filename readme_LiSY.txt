Note:
contact_offset: 0.04


low-Level
1-下载模型权重：放在 ./low-level/weights 目录下

2-low模式运行： 
conda activate b1z1
cd ./low-level/legged_gym/scripts
可以跑：
python play.py --exptid test --task b1z1 --proj_name b1z1-low --checkpoint 38000 --observe_gait_commands

go2训练：
python train.py --headless --exptid new_go2_0922 --proj_name b1z1-low --task b1z1 --sim_device cuda:0 --rl_device cuda:0 --observe_gait_commands                       
python play.py --exptid new_go2_0922 --task b1z1 --proj_name b1z1-low --checkpoint 8000 --observe_gait_commands



cd /home/lisy/Work/MySpace/005_WBC/visual_wholebody/high-level
This is how I trained the policy:
    python train_multistate.py --rl_device "cuda:0" --sim_device "cuda:0" --timesteps 500 --headless --task B1Z1PickMulti --experiment_dir b1-pick-multi-teacher-test --wandb --wandb_project "b1-pick-multi-teacher" --wandb_name "LiSY_HighLevel" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick  --debug
This is how I played the trained policy:
    python play_multistate.py --task B1Z1PickMulti --checkpoint "b1-pick-multi-teacher-test/LiSY_HighLevel/checkpoints/agent_500.pt" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick
I have not trained the policy with that much timesteps yet, since I wanted to test out the visualization first.


训练high-level:
1- 配置低层模型路径：在/home/lisy/Work/MySpace/005_WBC/visual_wholebody/high-level/data/cfg/b1z1_pickmulti.yaml
找到配置项里关于低层模型（low-level policy）的路径，把你已经训练好的低层权重（如 model_38000.pt）填进去。

2. 训练高层 teacher 策略，进入 high-level 目录，运行：
cd /home/lisy/Work/MySpace/005_WBC/visual_wholebody/high-level
python train_multistate.py --rl_device "cuda:0" --sim_device "cuda:0" --timesteps 500 --headless --task B1Z1PickMulti --experiment_dir b1-pick-multi-teacher-test --wandb --wandb_project "b1-pick-multi-teacher" --wandb_name "LiSY_HighLevel" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick


根据 high-level 目录下的 README，你可以按以下步骤运行高层策略：
1. 配置低层模型路径
打开 high-level 的配置文件（如 b1z1_pickmulti.yaml）。
找到配置项里关于低层模型（low-level policy）的路径，把你已经训练好的低层权重（如 model_38000.pt）填进去。
2. 训练高层 teacher 策略
进入 high-level 目录，运行：
cd /home/lisy/Work/MySpace/005_WBC/visual_wholebody/high-level
python train_multistate.py --rl_device "cuda:0" --sim_device "cuda:0" --timesteps 500 --headless --task B1Z1PickMulti --experiment_dir b1-pick-multi-teacher-test --wandb --wandb_project "b1-pick-multi-teacher" --wandb_name "LiSY_HighLevel" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick  --debug

3. 测试 teacher 策略
训练完成后，运行：
python play_multistate.py --task B1Z1PickMulti --checkpoint "b1-pick-multi-teacher-test/LiSY_HighLevel/checkpoints/agent_500.pt" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick

4. 训练 vision-based student 策略
python train_multi_bc_deter.py --headless --task B1Z1PickMulti --rl_device "cuda:0" --sim_device "cuda:0" --timesteps 60000 --experiment_dir "b1-pick-multi-stu" --wandb --wandb_project "b1-pick-multi-stu" --wandb_name "your_student_run" --teacher_ckpt_path "b1-pick-multi-teacher-test/LiSY_HighLevel/checkpoints/agent_60000.pt" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick  --debug


5. 测试 student 策略
python play_multi_bc_deter.py --task B1Z1PickMulti --checkpoint "b1-pick-multi-stu/your_student_run/checkpoints/agent_60000.pt"
python play_multi_bc_deter.py --task B1Z1PickMulti --checkpoint "b1-pick-multi-stu/your_student_run/checkpoints/agent_60000.pt" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick

---------------------------- v2 ------------------------------------------------
2. 训练高层 teacher 策略
进入 high-level 目录，运行：
cd /home/lisy/Work/MySpace/005_WBC/visual_wholebody/high-level
python train_multistate.py --rl_device "cuda:0" --sim_device "cuda:0" --timesteps 10000 --headless --task B1Z1PickMulti --experiment_dir b1-pick-multi-teacher-test --wandb --wandb_project "b1-pick-multi-teacher_v2" --wandb_name "LiSY_HighLevel_v2" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick  --debug

3. 测试 teacher 策略
训练完成后，运行：
python play_multistate.py --task B1Z1PickMulti --checkpoint "b1-pick-multi-teacher-test/LiSY_HighLevel_v2/checkpoints/agent_10000.pt" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick


4. 训练 vision-based student 策略
python train_multi_bc_deter.py --headless --task B1Z1PickMulti --rl_device "cuda:0" --sim_device "cuda:0" --timesteps 60000 --experiment_dir "b1-pick-multi-stu-v2" --wandb --wandb_project "b1-pick-multi-stu-v2" --wandb_name "your_student_run-v2" --teacher_ckpt_path "b1-pick-multi-teacher-test/LiSY_HighLevel_v2/checkpoints/agent_10000.pt" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick  --debug


5. 测试 student 策略
python play_multi_bc_deter.py --task B1Z1PickMulti --checkpoint "b1-pick-multi-stu-v2/your_student_run-v2/checkpoints/agent_60000.pt"
python play_multi_bc_deter.py --task B1Z1PickMulti --checkpoint "b1-pick-multi-stu-v2/your_student_run-v2/checkpoints/agent_60000.pt" --roboinfo --observe_gait_commands --small_value_set_zero --rand_control --stop_pick

