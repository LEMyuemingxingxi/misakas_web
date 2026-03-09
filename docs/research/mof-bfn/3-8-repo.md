## 训练结果

<img src="./assets/image-20260308090551268.png" alt="image-20260308090551268" style="zoom:67%;" />

* 由训练的loss曲线来看，整体loss（棕色线）比源代码（绿色线）要更低一些，但是没想到测试效果非常不好。

```bash
#/home/liem/data/mofbfn/checkpoints/logs/mof-csp/dng_sg_condition/ckpt/epoch_745-step_213356-loss_13.7210.ckpt

#/home/liem/data/mofbfn/checkpoints/logs/mof-csp/dng_sg_condition/ckpt/epoch_447-step_128128-loss_13.9678.ckpt

  
export LD_LIBRARY_PATH=/home/liem/.local/share/mamba/envs/mof-bfn/lib:$LD_LIBRARY_PATH

PYTHONPATH=. python -m experiments.infer \
  experiment.wandb.name=infer_dng_test \
  inference.ckpt_path=/home/liem/data/mofbfn/checkpoints/logs/mof-csp/dng_sg_condition/ckpt/epoch_447-step_128128-loss_13.9678.ckpt \
  inference.inference_dir=/home/liem/MOF-BFN/infer_results_test_3_8_2
  
  

python -m postprocess.assemble \
  --res_path /home/liem/MOF-BFN/infer_results_test_3_8_2/raw_results.pt \
  --bb_z_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_z.pt \
  --bb_blocks_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_blocks_processed.lmdb \
  --max_process 500

  
  # 检查连通性 (是否形成了完整的 3D 骨架)
python -m evaluation.connect_check \
  --res_path /home/liem/MOF-BFN/infer_results_test_3_8/raw_results.pt \
  --bb_z_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_z.pt \
  --bb_blocks_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_blocks_processed.lmdb \
  --max_process 500
  
  #有效性检查 (Validity Check)
  python -m evaluation.validity_check \
  --res_path /home/liem/MOF-BFN/infer_results_test_3_8/samples \
  --max_process 500
  
```



![image-20260308191202130](./assets/image-20260308191202130.png)

* 同样的指令，修改后竟然需要十几倍的时间来做？

![image-20260308192658424](./assets/image-20260308192658424.png)

最后连通性为0？？？
