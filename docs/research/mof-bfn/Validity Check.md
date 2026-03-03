## Validity Check

### TRUE

 'has_carbon',

 'has_hydrogen',

 'has_metal',

'is_porous':

* **有空隙（void）**
* 空隙不是封闭的死腔
* 空隙在 PBC(Principal Channel Bottleneck) 下是连通的
* 空隙尺寸 ≥ 探针半径（通常模拟 He / N₂）

### FALSE

* 几何性质合理

 'has_atomic_overlaps',

* 化学性质合理

 'has_overcoordinated_c',

 'has_overcoordinated_n',

 'has_overcoordinated_h',

 'has_undercoordinated_c',

 'has_undercoordinated_n',

 'has_undercoordinated_rare_earth',

 'has_lone_molecule',

* 经验化学先验

 'has_high_charges',（金属价态过高）

 'has_suspicicious_terminal_oxo',(孤立的 terminal M=O，在MOF中少见)

 'has_undercoordinated_alkali_alkaline',(碱金属、碱土金属配位数是否过低)

 'has_geometrically_exposed_metal'(金属中心是否在空间上 **明显暴露**)

> [!NOTE]
>
> exposed 是几何视角，undercoordinated 是化学视角





1D / 2D MOF 也是合法结构，"has_3d_connected_graph"直接跳过

```python
all_checks = []
for k, v in desc.items():
    if type(v) == bool:
        if k == "has_3d_connected_graph":
            continue
        if k in ["has_carbon", "has_hydrogen", "has_metal", "is_porous"]:
            all_checks.append(int(v))
        else:
            all_checks.append(int(not v))
desc["all_checks"] = np.all(all_checks)
```


