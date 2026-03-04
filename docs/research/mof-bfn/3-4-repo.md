

## “炼丹”速度迎来重大突破

之前我不断尝试训练指令时留下大量僵尸进程在GPU中，导致了训练一个epoch要10分钟，这次我清理干净之后再运行，速度显著变快了！！！！！！

![image-20260303221304319](./assets/image-20260303221304319.png)



## 训练完成

### 训练过程loss图示

前半段

![image-20260304103240477](./assets/image-20260304103240477.png)

后半段

![image-20260304103157425](./assets/image-20260304103157425-1772591587621-3-1772592678041-5.png)

与论文提供的chpt对比

![image-20260302191704618](./assets/image-20260302191704618.png)

### 训练过程 Loss 曲线分析

* **收敛瓶颈**：`val_lattice_loss`（晶格 Loss）在训练中后期进入平原期，波动极小。这解释了为什么模型倾向于预测“安全”的平均体积，因为这样可以获得稳定的低 Loss，但失去了对极端值的敏感度。

* **异常跳变**：`val_type_loss` 在后期出现剧烈刺状跳变，暗示模型在处理某些复杂配体（Building Blocks）的类型识别时存在震荡。

### 选择val_loss最小的ckpt以及最后一个ckpt进行测试

![image-20260304103132087](./assets/image-20260304103132087-1772591584007-1.png)

```bash

#/home/liem/data/mofbfn/checkpoints/logs/mof-csp/mof_dng_cif_200_inf/ckpt/epoch_413-step_118404-loss_13.4189.ckpt

#/home/liem/data/mofbfn/checkpoints/logs/mof-csp/mof_dng_cif_200_inf/ckpt/epoch_477-step_136708-loss_13.5524.ckpt


# 设置环境变量以防库冲突
export LD_LIBRARY_PATH=/home/liem/.local/share/mamba/envs/mof-bfn/lib:$LD_LIBRARY_PATH

PYTHONPATH=. python -m experiments.infer \
  experiment.wandb.name=infer_dng_test \
  inference.ckpt_path=/home/liem/data/mofbfn/checkpoints/logs/mof-csp/mof_dng_cif_200_inf/ckpt/epoch_477-step_136708-loss_13.5524.ckpt \
  inference.inference_dir=/home/liem/MOF-BFN/infer_results_test_3_4_last_ckpt
  
  

python -m postprocess.assemble \
  --res_path /home/liem/MOF-BFN/infer_results_test_3_4_last_ckpt/raw_results.pt \
  --bb_z_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_z.pt \
  --bb_blocks_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_blocks_processed.lmdb \
  --max_process 1000
  
  # 检查连通性 (是否形成了完整的 3D 骨架)
python -m evaluation.connect_check \
  --res_path /home/liem/MOF-BFN/infer_results_test_3_4_last_ckpt/raw_results.pt \
  --bb_z_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_z.pt \
  --bb_blocks_path /home/liem/data/mofbfn/datasets/dng/bb_emb_space_tot_blocks_processed.lmdb \
  --max_process 1000
  
  #有效性检查 (Validity Check)
  python -m evaluation.validity_check \
  --res_path /home/liem/MOF-BFN/infer_results_test_3_4_last_ckpt/samples \
  --max_process 1000
  

```

![image-20260304104407655](./assets/image-20260304104407655.png)

![image-20260304112154821](./assets/image-20260304112154821.png)

![image-20260304113649543](./assets/image-20260304113649543.png)

### validity效果对比

| **检查项 (Metric)**                              | **你的结果** | **论文最佳值 (Bold)** | **论文对比值** | **趋势建议**            |
| ------------------------------------------------ | ------------ | --------------------- | -------------- | ----------------------- |
| **has_carbon $\uparrow$**                        | **1.000**    | 1.000                 | 1.000          | 完美持平                |
| **has_hydrogen $\uparrow$**                      | 0.981        | 0.975                 | **0.989**      | 表现优异                |
| **has_atomic_overlaps $\downarrow$**             | 0.316        | **0.115**             | 0.259          | **偏高** (原子重叠较多) |
| **has_overcoordinated_c $\downarrow$**           | 0.464        | **0.193**             | 0.365          | **偏高** (碳配位数异常) |
| **has_overcoordinated_n $\downarrow$**           | 0.110        | 0.049                 | **0.047**      | **偏高**                |
| **has_overcoordinated_h $\downarrow$**           | 0.394        | **0.180**             | 0.342          | **偏高**                |
| **has_undercoordinated_c $\downarrow$**          | 0.205        | **0.182**             | 0.248          | 接近最佳水平            |
| **has_undercoordinated_n $\downarrow$**          | 0.164        | 0.154                 | **0.126**      | 较接近                  |
| **has_undercoordinated_rare_earth $\downarrow$** | 0.000        | 0.000                 | 0.000          | 完美持平                |
| **has_metal $\uparrow$**                         | **1.000**    | 1.000                 | 1.000          | 完美持平                |
| **has_lone_molecule $\downarrow$**               | 0.242        | **0.186**             | 0.437          | 优于对比值              |
| **has_high_charges $\downarrow$**                | 0.161        | **0.069**             | 0.144          | **偏高**                |
| **has_suspicious_terminal_oxo $\downarrow$**     | 0.000        | 0.000                 | 0.001          | 表现完美                |
| **has_undercoordinated_alkali... $\downarrow$**  | 0.098        | 0.027                 | **0.001**      | **偏高**                |
| **has_geometrically_exposed_metal $\downarrow$** | 0.453        | **0.304**             | 0.350          | **偏高**                |
| **all_checks (通过率) $\uparrow$**               | 0.167        | **0.350**             | 0.148          | **中等**                |


```py
  #晶胞大小分布
/home/liem/.local/share/mamba/envs/mof-bfn/bin/python - <<'PY'
import glob
import numpy as np

cifs = sorted(glob.glob('/home/liem/MOF-BFN/infer_results_test_3_4_last_ckpt/samples/pred_*.cif'))
print('cif_count', len(cifs))

vols = []
bad = []

# Prefer pymatgen for robust CIF parsing
try:
    from pymatgen.core.structure import Structure
except Exception as e:
    raise RuntimeError(f'pymatgen import failed: {e}')

for p in cifs:
    try:
        s = Structure.from_file(p)
        vols.append(float(s.lattice.volume))
    except Exception as e:
        bad.append((p, str(e)))

vols = np.asarray(vols, dtype=np.float64)
print('parsed', vols.size, 'failed', len(bad))
if bad:
    print('first_fail', bad[0][0], bad[0][1][:200])

if vols.size:
    print('mean', vols.mean(), 'std', vols.std(), 'min', vols.min(), 'max', vols.max())
    for p in [0,1,5,25,50,75,95,99,100]:
        print(f'p{p:>3}', np.percentile(vols, p))
    logv = np.log10(vols)
    print('log10(V) p1/p50/p99', np.percentile(logv,1), np.percentile(logv,50), np.percentile(logv,99))

    # plotly html
    try:
        import plotly.express as px
        fig = px.histogram(x=logv, nbins=80, labels={'x':'log10(pred_volume)'}, title='Generated samples: log10(pred_vol)')
        out = '/home/liem/MOF-BFN/infer_results_test_3_4_last_ckpt/samples/pred_log10vol_hist.html'
        fig.write_html(out)
        print('saved', out)
    except Exception as e:
        print('plotly failed/absent:', e)
PY
```

训练结果分布

![image-20260304112639049](./assets/image-20260304112639049.png)

![image-20260304112742007](./assets/image-20260304112742007.png)

train data 分布 

![image-20260303104401118](./assets/image-20260303104401118.png)

![image-20260303105125221](./assets/image-20260303105125221.png)

### 晶胞大小分布对比

通过观察 $log_{10}(V)$ 的分布图和分位数（p1, p50, p99）：

- **低端匹配较好**：$log_{10}(V)$ 的 p1 预测值为 2.88，真实值为 2.90，这说明模型对于小体积 MOF 的识别比较准确。
- **中位数偏移**：p50 预测值为 3.41，真实值为 3.53。这意味着一半以上的结构被预测得比实际小。
- **高端严重“断层”**：
  - 真实数据的 $log_{10}(V)$ 高端延伸到了 **5.0** 以上。
  - 预测数据的 $log_{10}(V)$ 在 **4.1** 之后几乎消失，最高仅到约 4.6。
  - 这表明模型完全丢失了对**超大孔径/体积**结构的预测能力。

### 总结

* 可能是因为我的惩罚过重，导致模型过于保守，不敢预测大晶胞
* 有没有可能是因为训练轮数不够多？因为论文给的ckpt是两千多个epoch
