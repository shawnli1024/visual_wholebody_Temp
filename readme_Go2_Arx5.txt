go2+Arx5配置说明：
1-/home/lisy/Work/MySpace/005_WBC/undergraduate/go2arx5/low-level/legged_gym/envs/manip_loco/b1z1_config_go2arx5.py  替换原来的 b1z1_config.py

2-将 /home/lisy/Work/MySpace/005_WBC/undergraduate/go2arx5/low-level/resources/robots/ 下的 arx5p2Go1 go2 放到原工程下

low-policy：
python train.py --headless --exptid new_go2_0922 --proj_name b1z1-low --task b1z1 --sim_device cuda:0 --rl_device cuda:0 --observe_gait_commands                       
python play.py --exptid new_go2_0922 --task b1z1 --proj_name b1z1-low --checkpoint 8000 --observe_gait_commands
