# 5模态对齐数据集完整列表（1种）

> 排序：条件类型 → 标注来源 → 尺寸(≥512²优先) → 分辨率 → 数量
> 裁剪：n = floor((S-512)/256)+1, 50%重叠

### 1. RGB↔NIR↔MS↔PAN↔SAR（2个数据集，总计 ~254K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                                                       | 原数量 | 裁剪       | 裁剪后 |
| ---- | --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------ | ---------- | ------ |
| ⭐1   | SpaceNet 6 MSAW | building polygons: G+Z(地理叠加，荷兰政府地籍数据+团队人工QC补漏删错); building height: G(地理叠加，从AHN LiDAR DSM自动提取匹配至每个polygon) | SAR/PAN/RGB/NIR: 900²@0.5m; MS: 225²@2.0m                    | 3K     | n=2, 2×2=4 | 13K    |
| 2    | FLAIR-HUB       | segmap(19类LULC): Z(专家标注); LPIS农田边界; DEM; cloud-free | RGB: 512²@0.2m; NIR: 512²@0.2m; PAN: 256²@0.4m; MS(SPOT): 64²@1.6m; SAR(S1): 10²@10.24m | 241K   | 原生尺寸   | 241K   |

**说明**：

**SpaceNet 6 MSAW**：

- SAR(Capella)↔PAN(WV-2)精对齐@0.5m，同900²px
- RGB/NIR为pan-sharpened产物，与SAR/PAN同像素网格900²@0.5m
- MS(原始4波段@2.0m,225²)与SAR/PAN/RGB/NIR同覆盖450m×450m，不同分辨率但同覆盖区域算对齐
- **5模态**：SAR↔PAN↔RGB↔NIR(精对齐) + MS(粗对齐，同覆盖)

**FLAIR-HUB**：

- 所有模态覆盖相同102.4m×102.4m地面区域，地理空间对齐
- **RGB**：来自AERIAL_RGBI的R/G/B三波段，512²@0.2m
- **NIR**：来自AERIAL_RGBI的近红外波段，512²@0.2m
- **PAN**：AERIAL-RLT_PAN历史航拍灰度图，256²@0.4m
- **MS**：SPOT_RGBI四波段，64²@1.6m（⚠️太小，无法裁剪到128²）
- **SAR**：SENTINEL1上升/下降轨道，10²@10.24m（⚠️太小，无法裁剪到256²）
- **问题**：MS和SAR模态像素尺寸太小，无法用于ControlNet任务
- **可用模态**：RGB+NIR+PAN（3模态可用），MS和SAR需放弃或特殊处理