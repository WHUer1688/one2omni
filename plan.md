# 数据集构建方案（RGB/NIR/SAR 256², PAN 512², MS 128²）

## 构建策略总结

构建多模态对齐数据集，目标**10M-100M样本对**，用于ControlNet式任务（prompt + 一个模态 → 另一个模态）。5种目标模态：RGB、NIR、MS、PAN、SAR。

**裁剪方案**：从大图裁剪至目标尺寸（RGB/NIR/SAR 256²、PAN 512²、MS 128²），同tile各模态裁剪相同空间位置保证对齐。原生≥256²的数据集直接裁剪无插值；原生128²的数据集通过2×插值至256²（名义GSD变小，有效分辨率不变）。GSD>10m或插值>4×的数据集舍弃。

**筛选策略**：优先选择模态数多的对齐数据集（5模态>4模态），条件标签优先segmap/instance mask/bbox（类别多优先），排除二值segmap（类别过少）和云覆盖严重的数据集（需筛选无云/少云样本）。

## 不足

- SpaceNet 3有模态PAN，但是条件是road centerline(7类)
- 方案选择：256²@10m的MS分成128，并且要和256²@10m的RGB对齐
  - 法一：**MS降低分辨率到20m**
  - 法二：先将256²@10m的RGB插值扩大到512²@5m，此时和256²@10m的MS对齐，然后前者裁剪出256，后者裁剪128出来，得到256²@5m的RGB和128²@10m的MS对齐

## 目录

- [裁剪后5模态对齐（1个数据集）](#裁剪后5模态对齐1个数据集34k-tiles) — SpaceNet 6
- [裁剪后4模态对齐 RGB/NIR/MS/SAR](#裁剪后4模态对齐rgbnirmssar)
  - [第一档：原生≥256²，直接裁剪（11个数据集）](#第一档原生256²直接裁剪11个数据集)
  - [第二档：可插值放大（4个数据集）](#第二档可插值放大4个数据集)
  - [第三档：不使用（7个数据集）](#第三档不使用7个数据集)
- [裁剪后4模态对齐 RGB/NIR/MS/PAN（5个数据集）](#裁剪后4模态对齐rgbnirmspan5个数据集)
- [总结](#总结)
- [数据集下载链接](#数据集下载链接)
- [实现细节](#实现细节)
  - [1. 裁剪参数与边缘处理](#1-裁剪参数与边缘处理)
  - [2. 插值算法](#2-插值算法)
  - [3. 数据格式与目录结构](#3-数据格式与目录结构)
  - [4. 条件标签统一类别映射](#4-条件标签统一类别映射)
  - [5. 云筛选实现](#5-云筛选实现)
  - [6. 对齐验证](#6-对齐验证)
  - [7. 存储需求估算](#7-存储需求估算)
  - [8. 处理优先级与工作流](#8-处理优先级与工作流)

------

> **目标尺寸**：RGB/NIR/SAR → 256²，PAN → 512²，MS → 128²
> **对齐定义**：同一tile内各模态裁剪后覆盖相同地面区域
> **插值经验**：1-2×✅、2-4×⚠️（名义GSD变小，有效分辨率不变）、>4×❌

------

## 裁剪后5模态对齐（1个数据集，~3.4K tiles）

> 同一tile内，裁剪/插值至目标尺寸后，RGB/NIR/SAR/PAN/MS五模态地理空间对齐
> FLAIR-HUB因SAR 10²→256²（25.6×）不可接受，无法满足5模态对齐

| 排名 | 数据集          | 原生尺寸                                  | RGB/NIR 256²        | PAN 512²           | MS 128²            | SAR 256²        | patches/tile(各模态)        | 裁剪后数据量(估) | 条件(来源)                                                   |
| ---- | --------------- | ----------------------------------------- | ------------------- | ------------------ | ------------------ | --------------- | --------------------------- | ---------------- | ------------------------------------------------------------ |
| ⭐1   | SpaceNet 6 MSAW | SAR/PAN/RGB/NIR: 900²@0.5m; MS: 225²@2.0m | 900²→256² ✅(5×5=25) | 900²→512² ✅(2×2=4) | 225²→128² ✅(2×2=4) | 900²→256² ✅(25) | 25/4/4/25（取4组对应patch） | ~3.4K×4=~13.6K   | **✅选用: building polygons**: Z(人工标注); building height→VLM上下文 |

**说明**：

- SpaceNet 6：PAN@0.5m裁4个512² patch，MS@2.0m裁4个128² patch，RGB/NIR/SAR@0.5m裁25个256² patch，取PAN/MS的4个位置对应RGB/NIR/SAR的4个patch → 每tile4组对应patch
- FLAIR-HUB：SAR 10²@10.24m→256²需25.6×插值（❌），放弃5模态对齐；仅RGB/NIR/PAN/MS可用（见PAN补充节）

------

## 裁剪后4模态对齐（RGB/NIR/MS/SAR）

> S2提供RGB/NIR/MS，S1提供SAR；同一tile内裁剪后四模态地理空间对齐

### 第一档：原生≥256²，直接裁剪（11个数据集）

| 排名 | 数据集                           | 原生尺寸                                                     | RGB/NIR 256² | MS 128²           | SAR 256² | patches/tile(各模态)   | 裁剪后数据量(估)                                             | 条件(来源)                                                   |
| ---- | -------------------------------- | ------------------------------------------------------------ | ------------ | ----------------- | -------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ⭐1   | SatlasPretrain                   | 512²@10m                                                     | ✅ 3×3=9      | ✅ 7×7=49          | ✅ 9      | 9/49/9（取9组对应）    | ~856K×9=~7.7M                                                | **✅选用: segmap(137类)**: Z+G+C; 其余(point/polygon/polyline/class/regression/properties)→VLM上下文; 全量35.8TB(S2+NAIP+L8/9+S1)，按需仅S1+S2+标签≈9.6TB |
| ⭐2   | M3LEO                            | 448²@10m                                                     | ✅ 2×2=4      | ✅ 6×6=36          | ✅ 4      | 4/36/4（取4组对应）    | ~17M×4=~68M                                                  | **✅选用: WorldCover segmap(11类)**448²: G; 其余(DEM/veg/biomass/built-up)→VLM上下文; 6区域:china\~4.85M/conus\~2.89M/europe\~2.81M/pakin\~2.55M/middleeast\~2.9M/southamerica~1.4M; ~17M tiles,18.73TB; S1/S2/LC/DEM/veg精细对齐448²，biomass/built-up粗分辨率存储需上采样 |
| ⭐3   | CloudLULC-Set(需筛选无云/少云)   | 512²@10m                                                     | ✅ 9          | ✅ 49              | ✅ 9      | 9/49/9                 | ~40K×9=~360K（筛选后数量待定）                               | **✅选用: segmap(6类: 水体/森林/草地/裸地/建成区/农田)**: Z(人工标注); 云覆盖信息→VLM上下文; 需筛选低云样本(≤20%) |
| 4    | TerraMesh                        | 264²@10m                                                     | ✅ 1          | ✅ 4               | ✅ 1      | 1/4/1                  | 9M+×1=~9M+                                                   | **✅选用: LULC segmap(10类)**: G; 其余(DEM/NDVI/S2L1C/S1RTC/cloud_mask)→VLM上下文; Zarr Zip格式; 85%S1/S2时间对齐1周内 |
| 5    | SEN12MS                          | 256²@10m                                                     | ✅ 1          | ✅ 4               | ✅ 1      | 1/4/1                  | ~180K×1=~180K                                                | **✅选用: MODIS segmap(33类)**: G(500m分辨率，质量较低); metadata(季节/位置CSV)→VLM上下文 |
| 6    | OlmoEarth                        | 256²@10m                                                     | ✅ 1          | ✅ 1(原生128²@20m) | ✅ 1      | 1/1/1                  | ~285K×1=~285K                                                | **✅选用: WorldCover segmap(11类)**: G(地理叠加:ESA WorldCover 2021); CDL作物(~100类,仅美国本土): G→第二条件; SRTM/OSM/ERA5/冠层高度/WorldCereal→VLM上下文; 月度时序×12期(360天)可扩充至~3.4M; 全量16.15TB/按需≈4.6TB; 论文arXiv:2511.13655 |
| 7    | MADS                             | S1/S2: 1371×888@10m(全城场景); HSI: @2.2m; DSM: @0.25m; 无PAN | ✅ 5×3=15     | ✅ 10×6=60         | ✅ 5×3=15 | 15/60/15（取15组对应） | 数量暂未知，单scene约15组patch；37GB下载量含HSI/DSM/模拟产品等大量辅助数据 | **✅选用: segmap(6类LC: pavement/low vegetation/soil/tree/roof/water)**: Z(人工标注，仅3子区域); 其余(HSI 416波段/DSM 0.25m/端元光谱859条/OSM GIS/S2eteS_S2/EeteS_EnMAP)→VLM上下文; Augsburg全城，121.7km²，2018-05-07同日采集 |
| 8    | LuojiaSET-OSFCR(需筛选无云/少云) | 256²@10m                                                     | ✅ 1          | ✅ 4               | ✅ 1      | 1/4/1                  | ~20K×1=~20K（筛选后数量待定）                                | **✅选用: segmap(LC分类)**: P(QC); 云覆盖信息→VLM上下文; 需筛选低云样本 |
| 9    | HunanMultimodal                  | 256²@10m                                                     | ✅ 1          | ✅ 4               | ✅ 1      | 1/4/1                  | ~500×1=~500                                                  | **✅选用: segmap(7类LULC)**: Z(人工标注); DEM→VLM上下文       |
| 10   | DFC20/DFC2020                    | 256²@10m                                                     | ✅ 1          | ✅ 4               | ✅ 1      | 1/4/1                  | ~6K×1=~6K                                                    | **✅选用: segmap(10类LULC)**: G(MODIS MCD12Q1叠加)            |
| 11   | MultiSenGE                       | 256²@10m                                                     | ✅ 1          | ✅ 4               | ✅ 1      | 1/4/1                  | ~8K×1=~8K                                                    | **✅选用: segmap(10类LULC)**: G(OCSGE2-GEOGRANDEST政府LULC数据库，原14类合并为10类); 多时相S1/S2 |

> 有个顾虑：256²@10m的MS分成128，并且要和256²@10m的RGB对齐，**需要降低分辨率到20m**；有种办法：先将256²@10m的RGB插值扩大到512²@5m，此时和256²@10m的MS对齐，然后前者裁剪出256，后者裁剪128出来，得到256²@5m的RGB和128²@10m的MS对齐

### 第二档：可插值放大（4个数据集）

> 原生128²@10m覆盖1280m×1280m；插值到256²后名义GSD变为5m（有效分辨率仍~10m），仍覆盖1280m×1280m；与MS(128²@10m)覆盖相同地面区域，对齐✅；1个MS像素≈插值RGB的2×2像素

| 排名 | 数据集        | 原生尺寸                                                     | RGB/NIR 256²          | MS 128²               | SAR 256²           | patches/tile | 裁剪后数据量(估) | 条件(来源)                                                   | 插值分析                                                     |
| ---- | ------------- | ------------------------------------------------------------ | --------------------- | --------------------- | ------------------ | ------------ | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | MMEarth       | 128²@10m                                                     | **2×** ✅              | ✅ 1(无插值)           | **2×** ✅           | 1/1/1        | ~1.2M×1=~1.2M    | **✅选用: WorldCover segmap(11类)**: G; 其余(DEM/冠层高度/Biome/ERA5/DW LandCover)→VLM上下文 | 名义GSD 10m→5m（有效分辨率仍~10m），覆盖1280m²不变；MS 128²@10m无插值 |
| 2    | BigEarthNet   | S1=128²@10m(SAR 2ch VV/VH); S2=120²@10m(RGB+NIR+MS 12ch B01-B12); 无PAN | 120²→256² **2.13×** ⚠️ | 120²→128² **1.07×** ✅ | 128²→256² **2×** ✅ | 1/1/1        | 549,488×1=~549K  | **✅选用: segmap(19类多标签+像素级参考图)**: G; metadata→VLM上下文 | 名义GSD改变（S2 10m→4.7m，S1 10m→5m），有效分辨率仍~10m；无PAN |
| 3    | CerraData-4MM | 128²@10m                                                     | **2×** ✅              | ✅ 1(无插值)           | **2×** ✅           | 1/1/1        | 30,322×1=~30K    | **✅选用: segmap L2(14子类)+segmap L1(7大类)**: P(QC)(TerraClass DL辅助+专家验证) | 名义GSD 10m→5m（有效分辨率仍~10m），覆盖1280m²不变；MS 128²@10m无插值 |

### 第三档：不使用（8个数据集）

| 数据集              | 原生尺寸          | 不使用原因                               |
| ------------------- | ----------------- | ---------------------------------------- |
| ~~S1S2-Water~~      | 256²@10m          | 条件仅为水体二值segmap，类别过少         |
| ~~Sen1Floods11~~    | 512²@10m          | 条件仅为水体/洪水二值segmap，类别过少    |
| ~~Plot-Rice~~       | 256²@10m          | 条件仅为水稻/非水稻二值segmap，类别过少  |
| ~~Sen12Landslides~~ | 128²@10m          | 条件仅为滑坡二值segmap，类别过少         |
| ~~CloudSEN12~~      | 509²@10m          | 条件标签是云mask，不是LC segmap，无用    |
| ~~MajorTOM-Core~~   | S2:1068², S1:200² | 仅有cloud mask标注，不适合ControlNet任务 |
| ~~So2Sat~~          | 32²@10m           | RGB/NIR/SAR需8×插值❌                     |
| ~~TalloS~~          | 64²@10m           | RGB/NIR/SAR需4×插值❌                     |

------

## 裁剪后4模态对齐（RGB/NIR/MS/PAN，5个数据集）

> 同一tile内裁剪后RGB/NIR/MS/PAN四模态地理空间对齐

| 排名 | 数据集     | 原生尺寸                                      | RGB/NIR 256²       | PAN 512²                | MS 128²                | patches/tile(各模态)  | 裁剪后数据量(估) | 条件(来源)                                                   |
| ---- | ---------- | --------------------------------------------- | ------------------ | ----------------------- | ---------------------- | --------------------- | ---------------- | ------------------------------------------------------------ |
| ⭐1   | FLAIR-HUB  | RGB/NIR:512²@0.2m; PAN:256²@0.4m; MS:64²@1.6m | 512²→256² ✅(3×3=9) | 256²→**插值512²** ✅(2×) | 64²→**插值128²** ⚠️(2×) | 9/1/1（取9组对应）    | ~241K×9=~2.2M    | **✅选用: segmap(19类LC)**: Z(人工标注); 其余(23类作物/DSM/DTM)→VLM上下文 |
| ⭐2   | SpaceNet 4 | 900²@50cm                                     | ✅ 5×5=25           | ✅ 2×2=4                 | ✅ 13×13=169            | 25/4/169（取4组对应） | ~248K×4=~992K    | **✅选用: building polygons**: Z(人工标注)                    |
| ⭐3   | SpaceNet 5 | 650²@30cm                                     | ✅ 4×4=16           | ✅ 2×2=4                 | ✅ 9×9=81               | 16/4/81（取4组对应）  | ~25K×4=~100K     | **✅选用: building polygons + road centerline**: Z(人工标注); travel time→VLM上下文 |
| 4    | SpaceNet 2 | 650²@30cm                                     | ✅ 16               | ✅ 4                     | ✅ 81                   | 16/4/81               | ~24K×4=~96K      | **✅选用: building polygons**: Z(人工标注)                    |
| 5    | SpaceNet 3 | 650²@30cm                                     | ✅ 16               | ✅ 4                     | ✅ 81                   | 16/4/81               | ~59K×4=~236K     | **✅选用: road centerline(7类)**: Z(QA/QC); metadata(OSM)→VLM上下文 |

**FLAIR-HUB插值说明**：

- RGB/NIR：512²→256²裁剪，无插值 ✅
- PAN：256²@0.4m→512²，2×插值 ✅（名义GSD 0.4m→0.2m，有效分辨率仍~0.4m，清晰）
- MS：64²@1.6m→128²，2×插值 ⚠️（名义GSD 1.6m→0.8m，有效分辨率仍~1.6m，可用但细节无新增）
- SAR：10²@10.24m→256²，25.6×插值 ❌（基本无意义，放弃SAR模态）

------

## 总结

| 方案                              | 数据集数 | 裁剪后数据量(估) | 可用数据集                    |
| --------------------------------- | -------- | ---------------- | ----------------------------- |
| 裁剪后5模态对齐                   | 1        | ~13.6K           | SpaceNet6(全5)                |
| 裁剪后4模态(RGB/NIR/MS/SAR)第一档 | 11       | ~86.3M           | 原生裁剪，无插值              |
| 裁剪后4模态(RGB/NIR/MS/SAR)第二档 | 3        | ~1.76M           | MMEarth/BigEarthNet/CerraData |
| 裁剪后4模态(RGB/NIR/MS/PAN)       | 5        | ~3.6M            | FLAIR-HUB/SpaceNet 2/3/4/5    |

**关键结论**：

1. **对齐=裁剪后地理空间对齐**：同一tile各模态裁剪至目标尺寸后，相同索引patch覆盖相同地面区域
2. **FLAIR-HUB无法5模态对齐**：SAR 10²→256²需25.6×插值❌，仅RGB/NIR/PAN/MS四模态可用
3. **M3LEO是最大4模态数据集**：448²裁4组对应patch，17M×4=~68M
4. **256²大幅降低门槛**：11个数据集可直接裁剪（原512²仅5个）
5. **第二档3个数据集**：128²@10m原生通过2×插值至256²，名义GSD变为5m（有效分辨率仍~10m），覆盖范围仍为1280m²；与MS(128²@10m)对齐✅；1个MS像素≈插值RGB的2×2像素；Sen12Landslides因仅滑坡二值移至第三档
6. **PAN不丢弃**：FLAIR-HUB PAN 256²@0.4m通过2×插值至512²✅
7. **插值经验：1-2×✅、2-4×⚠️、>4×❌（如TalloS 4×、So2Sat 8×均放弃）
8. **MADS无PAN**：仅4模态对齐(RGB/NIR/MS/SAR)，37GB含HSI(416波段@2.2m)/DSM(0.25m)/模拟产品(S2eteS_S2/EeteS_EnMAP_10/30)/OSM GIS等大量辅助数据；Augsburg全城121.7km²，2018-05-07同日采集；segmap仅覆盖3个子区域(6类LC)；单scene约15组patch
9. **CloudSEN12已排除**：条件标签是云mask，不是LC segmap，无用

------

## 数据集下载链接

### S3已上传

| 数据集                           | S3路径                                            | 大小      | 备注           |
| -------------------------------- | ------------------------------------------------- | --------- | -------------- |
| CloudLULC-Set(需筛选无云/少云)   | /chy/One2Omni/datasets/bdwp/CloudLULC.rar         | 185.18 GB | 需筛选低云样本 |
| BigEarthNet                      | /chy/One2Omni/datasets/bdwp/BigEarthNet-v2.0/     | —         | 549K pairs     |
| LuojiaSET-OSFCR(需筛选无云/少云) | /chy/One2Omni/datasets/LuojiaSET-OSFCR.zip        | 37.70 GB  | 需筛选低云样本 |
| HunanMultimodal                  | /chy/One2Omni/datasets/HunanMultimodalDataset.rar | 1.72 GB   | 500 tiles      |
| SpaceNet 2/3/4/5/6               | /chy/One2Omni/datasets/SpaceNet/                  | —         |                |

### HuggingFace下载（huggingface-cli）

| 数据集             | 命令                                                         | HF链接                                                       | 备注                                                         |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SatlasPretrain     | `huggingface-cli download allenai/satlas-pretrain --include "dataset/*"` | —                                                            | 856K tiles；全量35.8TB，按需S1+S2+标签≈9.6TB，137类标签；该仓库为model类型（数据在dataset/子目录），加--repo-type dataset会401 |
| M3LEO/china        | `huggingface-cli download M3LEO/china --repo-type dataset`   | https://huggingface.co/datasets/M3LEO/china                  | ~4.85M tiles,4.71TB                                          |
| M3LEO/conus        | `huggingface-cli download M3LEO/conus --repo-type dataset`   | https://huggingface.co/datasets/M3LEO/conus                  | ~2.89M tiles,3.04TB                                          |
| M3LEO/europe       | `huggingface-cli download M3LEO/europe --repo-type dataset`  | https://huggingface.co/datasets/M3LEO/europe                 | ~2.81M tiles,3.68TB                                          |
| M3LEO/pakin        | `huggingface-cli download M3LEO/pakin --repo-type dataset`   | https://huggingface.co/datasets/M3LEO/pakin                  | ~2.55M tiles,2.57TB                                          |
| M3LEO/middleeast   | `huggingface-cli download M3LEO/middleeast --repo-type dataset` | https://huggingface.co/datasets/M3LEO/middleeast             | ~2.9M tiles,3.53TB                                           |
| M3LEO/southamerica | `huggingface-cli download M3LEO/southamerica --repo-type dataset` | https://huggingface.co/datasets/M3LEO/southamerica           | ~1.4M tiles,1.2TB                                            |
| TerraMesh          | `huggingface-cli download ibm-esa-geospatial/TerraMesh --repo-type dataset` | https://huggingface.co/datasets/ibm-esa-geospatial/TerraMesh | 9.1M pairs，17 TB                                            |
| MADS               | `huggingface-cli download torchgeo/mdas --repo-type dataset` | https://huggingface.co/datasets/torchgeo/mdas                | 37.2GB                                                       |
| DFC20/DFC2020      | `huggingface-cli download 125oii/dfc2020 --repo-type dataset` | https://huggingface.co/datasets/125oii/dfc2020               | 原始256²，20.7 GB                                            |
| OlmoEarth          | `huggingface-cli download allenai/olmoearth_pretrain_dataset --include "10_sentinel2_l2a_monthly/*" --include "10_sentinel1_monthly/*" --include "10_worldcover/*" --include "10_cdl/*"` | https://huggingface.co/datasets/allenai/olmoearth_pretrain_dataset | 285,288 samples；全量16.15TB（50GB tar/模态），按需(S2+S1+WorldCover+CDL)≈4.6TB |

### 直链下载（IDM/浏览器）

| 数据集        | 下载链接                                                     | 备注                          |
| ------------- | ------------------------------------------------------------ | ----------------------------- |
| MMEarth       | https://sid.erda.dk/sharelink/ChL1BoVEyH                     | 1.2M pairs，597GB，HDF5格式   |
| SEN12MS       | https://mediatum.ub.tum.de/1474000 (主) / https://www.kaggle.com/datasets/tapakahkun/sen12ms (备) | 180,662 groups                |
| MultiSenGE    | https://hal.science/hal-03776281                             | 8,157 patches                 |
| CerraData-4MM | https://www.kaggle.com/datasets/cerranet/cerradata-4mm       | 30,322 pairs，128²@10m，S1+S2 |

------

## 实现细节

### 1. 裁剪参数与边缘处理

**50%重叠裁剪参数**：

| 模态        | crop_size | stride | 原生尺寸→patches/tile                          | 说明              |
| ----------- | --------- | ------ | ---------------------------------------------- | ----------------- |
| RGB/NIR/SAR | 256       | 128    | 512²→3×3=9, 448²→2×2=4, 264²→1×1=1, 256²→1×1=1 | 50%重叠增加数据量 |
| PAN         | 512       | 256    | 900²→2×2=4, 256²→1×1=1                         | PAN分辨率=RGB×2   |
| MS          | 128       | 64     | 225²→2×2=4, 264²→2×2=4, 128²→1×1=1             | MS分辨率=RGB/2    |

**边缘patch处理**：保留贴边窗口（不丢弃），确保覆盖图像最右/最下侧。边缘不足crop_size的区域仍生成一个完整尺寸patch，不足部分用reflect padding填充。

**不同原生尺寸的对齐裁剪**：

- 同一tile内，各模态以**相同地面中心点**为参考裁剪
- 裁剪公式：`patch位置 = (地面中心坐标 - 原图左上角) / 像素尺寸`
- 例：SpaceNet 6的PAN(900²@0.5m)和MS(225²@2.0m)，取PAN的4个512² patch位置，对应MS裁4个128² patch，覆盖相同地面区域
- 裁剪后用**文件名记录坐标**：`{dataset}__{tile_id}__x{left}_y{top}.png`（参考sliceSkill命名规范）

### 2. 插值算法

| 场景                    | 算法           | 说明                                   |
| ----------------------- | -------------- | -------------------------------------- |
| 第二档128²→256²         | **双线性插值** | 计算快，质量够用，2×倍率下效果好       |
| FLAIR-HUB PAN 256²→512² | **双线性插值** | 同上                                   |
| FLAIR-HUB MS 64²→128²   | **双线性插值** | 同上                                   |
| 条件segmap              | **最近邻插值** | 保持类别整数值不变，避免插值产生中间值 |

**不使用双三次**：计算量大但质量提升微小，对遥感影像2×插值收益不大。

### 3. 数据格式与目录结构

**输出格式**：

- 影像：**TIFF**（保留地理元数据，多波段兼容）
- segmap：**PNG**（单通道，类别ID存储）
- bbox/polygon：**XML**（参考sliceSkill VOC格式）

**目录结构**：

```
DatasetRoot/
  RGB/
    {dataset}__{tile}__x{left}_y{top}.png
  NIR/
    {dataset}__{tile}__x{left}_y{top}.png
  MS/
    {dataset}__{tile}__x{left}_y{top}.png
  PAN/                    （如有PAN模态）
    {dataset}__{tile}__x{left}_y{top}.png
  SAR/
    {dataset}__{tile}__x{left}_y{top}.png
  Conditions/
    segmap/
      {dataset}__{tile}__x{left}_y{top}.png
    bbox/                 （如有bbox标注）
      {dataset}__{tile}__x{left}_y{top}.xml
  Metadata/
    index.json            # 入口索引 [{"image": "RGB/xxx.png", "segmap": "Conditions/segmap/xxx.png", ...}]
    cloud_stats.json      # 云覆盖统计（CloudLULC/LuojiaSET专用）
```

**命名规则**：`{source_dataset}__{tile_id}__x{left}_y{top}.png`，保证可追溯原图和位置。

### 4. 条件标签统一类别映射

不同数据集segmap类别不同，需建统一映射表（10类基础LULC）：

| 统一类别ID | 统一类别名 | SatlasPretrain | M3LEO(WorldCover) | CloudLULC-Set | TerraMesh(ESRI) | SEN12MS(MODIS) | MADS(6类segmap) | CerraData-4MM(L1/L2)         |
| ---------- | ---------- | -------------- | ----------------- | ------------- | --------------- | -------------- | --------------- | ---------------------------- |
| 0          | 水体       | Water          | Water             | 水体          | Water           | Water          | Water           | Wt(水体)                     |
| 1          | 森林       | Tree           | Trees             | 森林          | Trees           | Forest         | Tree            | Ab/Pr/Ft(乔木/多年生/林业)   |
| 2          | 草地       | Grassland      | Grassland         | 草地          | Grassland       | Grassland      | Low vegetation  | Pa(牧场)/Meadow              |
| 3          | 农田       | Cropland       | Crops             | 农田          | Crops           | Cropland       | —               | Ag(农业:T1/T1+/SP)           |
| 4          | 建成区     | Built          | Built             | 建成区        | Built           | Urban          | Roof/Pavement   | Bg/UA/OB(建筑/城市/其他建筑) |
| 5          | 裸地       | Bare           | Bare              | 裸地          | Barren          | Barren         | Soil            | Df(毁林)/Mg(采矿)/OU(其他)   |
| 6          | 湿地       | Wetland        | Wetland           | —             | Wetland         | Wetland        | —               | —                            |
| 7          | 苔原       | —              | Tundra            | —             | —               | —              | —               | —                            |
| 8          | 灌木       | Shrubland      | Shrubland         | —             | —               | —              | —               | —                            |
| 9          | 冰雪       | Snow/Ice       | Snow/Ice          | —             | —               | Snow/Ice       | —               | —                            |

**注意**：

- 不同数据集类别数不同（CloudLULC仅6类，M3LEO有11类），统一映射后保留原始segmap，统一映射作为辅助
- MADS segmap为手动标注的6类（pavement/low vegetation/soil/tree/roof/water），仅覆盖3个子区域
- CerraData-4MM有两级：L1(7类: Pa/Ab/Ag/Mg/Bg/Wt/OU)和L2(14类细分)，使用L1作统一映射
- SEN12MS标签为MODIS MCD12Q1(500m分辨率)，质量较低

### 5. 云筛选实现

| 数据集          | 云覆盖字段                               | 筛选方法                          | 预期保留率              |
| --------------- | ---------------------------------------- | --------------------------------- | ----------------------- |
| CloudLULC-Set   | Fmask 4.6生成的cloud_coverage_percentage | 筛选`cloud_coverage <= 20%`的样本 | ~20%（5个区间均匀分布） |
| LuojiaSET-OSFCR | 元数据中的cloud_cover字段                | 筛选`cloud_cover <= 20%`的样本    | 待验证                  |

**实现步骤**：

1. 读取数据集的cloud coverage元数据（JSON/CSV）
2. 筛选低云样本（cloud_coverage <= 20%）
3. 仅下载/处理筛选后的样本
4. 生成cloud_stats.json记录每个patch的云覆盖值

### 6. 对齐验证

裁剪后验证不同模态patch确实对齐：

**验证方法**：

1. 随机抽取10个patch组（同一tile的RGB/NIR/MS/SAR同位置patch）
2. 可视化叠加：将RGB与SAR边缘叠加，检查地理特征（道路、建筑边缘、河流）是否重合
3. 量化验证：计算互相关系数（NCC），理想值>0.8（不同模态亮度不同但边缘应一致）

**验证流程**：每个数据集裁剪后，运行验证脚本，生成验证报告（含可视化图）。

### 7. 存储需求估算

| 数据集         | 下载大小                                               | 裁剪后估算 | 说明                                                         |
| -------------- | ------------------------------------------------------ | ---------- | ------------------------------------------------------------ |
| M3LEO          | 18.73TB                                                | ~10TB      | 448²裁4组，17M×4=68M patches                                 |
| SatlasPretrain | 按需~9.6TB（S1+S2+标签；全量35.8TB，NAIP/Landsat不下） | ~80GB      | 512²裁9组，856K×9=7.7M patches                               |
| TerraMesh      | 17TB                                                   | ~5TB       | 264²裁1组，9.1M patches（7种模态，S2L2A 12ch为主）           |
| CloudLULC-Set  | 185GB                                                  | ~60GB      | 512²裁9组，筛选后约40K×9=360K                                |
| BigEarthNet    | ~80GB                                                  | ~30GB      | 120²/128²裁1组，549K patches                                 |
| MMEarth        | 597GB                                                  | ~200GB     | 128²裁1组，1.2M patches                                      |
| FLAIR-HUB      | 726GB                                                  | ~40GB      | 512²裁9组，241K×9=2.2M                                       |
| OlmoEarth      | 按需~4.6TB（S1+S2+WorldCover+CDL；全量16.15TB）        | ~100GB     | 256²原生直取，285K patches（时相扩充可至~3.4M）              |
| SpaceNet系列   | ~150GB                                                 | ~60GB      | 各城市合计                                                   |
| 其他           | ~50GB                                                  | ~20GB      | SEN12MS/DFC20/MultiSenGE等                                   |
| **总计**       | **~52TB**                                              | **~16TB**  | M3LEO（18.73TB）+ TerraMesh（17TB）+ Satlas按需（9.6TB）+ OlmoEarth按需（4.6TB）占大头；下载+裁剪后均需保留至验证完成 |

**注意**：M3LEO 18.73TB + TerraMesh 17TB + Satlas按需9.6TB + OlmoEarth按需4.6TB ≈ 50TB占大头，下载总量约52TB，裁剪后约16TB，共需约70TB临时空间。验证完成后可删除原始下载。M3LEO/TerraMesh/Satlas/OlmoEarth建议按区域/分片分批下载处理；Satlas与OlmoEarth均可只下S1+S2+标签子集。

### 8. 处理优先级与工作流

**处理顺序**（按数据量从大到小）：

1. **M3LEO**（68M patches）— 最大数据集，先处理
2. **SatlasPretrain**（7.7M patches）— 第二大数据集
3. **TerraMesh**（9.1M patches）— 第三大
4. **MMEarth**（1.2M patches）— 第二档，需插值
5. **BigEarthNet**（549K patches）— 第二档，需插值
6. **FLAIR-HUB**（2.2M patches）— 有PAN模态
7. **SpaceNet系列**（各城市）— 有PAN模态
8. **其他小数据集** — CloudLULC-Set/SEN12MS/DFC20/MultiSenGE/CerraData/MADS/HunanMultimodal/LuojiaSET/OlmoEarth

**每个数据集处理流程**：

1. 下载 → 2. 验证原生尺寸/波段 → 3. 云筛选（如有） → 4. 裁剪/插值 → 5. 对齐验证 → 6. 生成index.json → 7. 压缩存档

**数据量目标检查**：

- 当前估算总量：68M+7.7M+9.1M+1.2M+549K+2.2M+285K+其他 ≈ **89M patches**（Sen12Landslides已移至第三档）
- 目标10M-100M ✅，当前方案可达约89M，接近上限，无需补充数据集

------

裁剪后5模态对齐

SpaceNet 6 MSAW — 模态: SAR/PAN/RGB/NIR 900²@0.5m, MS 225²@2.0m | 条件: ✅building polygons + height

![image-20260908214845884](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908214845884.png)

![building footprints](https://miro.medium.com/max/500/1*x5VCNbYLjUmxjiLiT9jrYA.png)

裁剪后4模态对齐（RGB/NIR/MS/SAR）

SatlasPretrain — 模态: RGB/NIR/MS/SAR 512²@10m | 条件: ✅segmap(137类7标注类型)

![image-20260908222952317](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908222952317.png)

![Natural waterways](https://satlas-pretrain.allen.ai/c8ca6e4b9d8f934c48c24df648560351.jpg)

![Offshore wind turbines & vessels](https://satlas-pretrain.allen.ai/acd3a8745349c416222bc68504488563.jpg)

![Mine shafts](https://satlas-pretrain.allen.ai/1ef56b57c80d8c7a30819d9a4b787d2b.jpg)

M3LEO — 模态: SAR/S2/WorldCover/DEM/veg 448²@10m | 条件: ✅WorldCover segmap(11类)

![image/png](https://cdn-uploads.huggingface.co/production/uploads/665367aa9ccb17d96757c105/IyD7zNzQ6gzhYLrrnMhpF.png)

> Sentinel 1 GRD Amplitude：哨兵1号 SAR 雷达幅度图，表示地物对雷达信号的后向散射强度，可反映地表结构和粗糙度。
>
> Sentinel 1 Coherence：哨兵1号干涉相干性，表示两次 SAR 观测之间的相似程度；高相干说明地表变化较小，低相干可能表示植被、运动或地表变化。
>
> Sentinel 1 Interferograms：哨兵1号干涉图，由两次 SAR 影像的相位差得到，可用于测量地表形变、高程等信息。
>
> Sentinel 2 Multi-spectral：哨兵2号多光谱影像，同时记录可见光、近红外、短波红外等多个波段，用于识别植被、水体、建筑等地物。
>
> Biomass：生物量，表示一定区域内植被等生物物质的总量，常用于描述森林或植被生长状况。
>
> Vegetation %：植被覆盖率，表示一个区域中被植被覆盖的面积比例。
>
> Built-up Surface：建成区表面，表示建筑物、道路等人工建设区域的分布或覆盖程度。
>
> World Cover：全球土地覆盖分类图，将地表划分为森林、草地、水体、建筑、农田等类别。
>
> DEM：数字高程模型（Digital Elevation Model），表示地表各位置的海拔或地形高度。

CloudLULC-Set(需筛选无云/少云) — 模态: RGB/NIR/MS/SAR 512²@10m | 条件: ✅segmap(6类)

![image-20260908201854463](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908201854463.png)

TerraMesh — 模态: RGB/NIR/MS/SAR 264²@10m | 条件: ✅ESRI segmap(10类)

![Examples from TerraMesh](https://huggingface.co/datasets/ibm-esa-geospatial/TerraMesh/resolve/main/assets%2Fexamples.png)

SEN12MS — 模态: RGB/NIR/MS/SAR 256²@10m | 条件: ✅MODIS segmap(33类)

![image-20260908215834347](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908215834347.png)

MADS — 模态: SAR 10m/MS 10m/HSI 2.2m/DSM 0.25m/GIS | 条件: ✅segmap(6类LC，仅3子区域)

![https://essd.copernicus.org/articles/15/113/2023/essd-15-113-2023-f01](https://essd.copernicus.org/articles/15/113/2023/essd-15-113-2023-f01-thumb.jpg)

![https://essd.copernicus.org/articles/15/113/2023/essd-15-113-2023-f06](https://essd.copernicus.org/articles/15/113/2023/essd-15-113-2023-f06-thumb.png)

LuojiaSET-OSFCR(需筛选无云/少云) — 模态: RGB/NIR/MS/SAR 256²@10m | 条件: ✅segmap(LC分类)

HunanMultimodal — 模态: RGB/NIR/MS/SAR 256²@10m | 条件: ✅segmap(7类LULC) + DEM

![image-20260908213017020](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908213017020.png)

DFC20/DFC2020 — 模态: RGB/NIR/MS/SAR 256²@10m | 条件: ✅segmap(10类LULC)

MultiSenGE — 模态: RGB/NIR/MS/SAR 256²@10m | 条件: ✅segmap(10类LULC，原14类合并)



![image-20260908212706459](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908212706459.png)

![image-20260908212331964](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908212331964.png)

MMEarth — 模态: RGB/NIR/MS/SAR 128²@10m | 条件: ✅WorldCover segmap(11类)

![image-20260908215520354](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20260908215520354.png)

> Aster-DEM Elevation：ASTER 数字高程模型，表示地表各位置的海拔高度和地形起伏。
>
> ESA WorldCover：欧洲航天局发布的全球土地覆盖分类数据，将地表划分为森林、农田、水体、建筑等类别。
>
> Dynamic World：基于 Sentinel-2 影像生成的动态全球土地覆盖数据，可反映不同时期地表覆盖类型及其变化。
>
> Canopy Height：冠层高度，表示森林或植被顶部相对于地面的高度，可用于描述植被结构和森林状况。

CerraData-4MM — 模态: RGB/NIR/MS/SAR 128²@10m | 条件: ✅segmap L2(14子类)+L1(7大类)

![dataset_git.png](https://github.com/ai4luc/CerraData-4MM/blob/main/visual_setup/dataset_git.png?raw=true)

FLAIR-HUB — 模态: RGBI 512²@0.2m, PAN 256²@0.4m, SPOT MS 64²@1.6m | 条件: ✅CoSIA segmap(15-19类LC)

![img](https://ignf.github.io/FLAIR/img/FLAIR-HUB_patches_1_VERTICAL.png)

SpaceNet 4 — 模态: RGB/NIR/PAN/MS 900²@0.5m | 条件: ✅building polygons

![img](https://spacenet.ai/wp-content/uploads/2019/01/SpaceNet-winners-300x298.jpg)

SpaceNet 5 — 模态: RGB/NIR/PAN/MS 1300²@0.3m | 条件: ✅building polygons + road centerline

![img](https://spacenet.ai/wp-content/uploads/2019/08/Las_Vegas_GIF4.gif)

SpaceNet 2 — 模态: RGB/NIR/PAN/MS 650²@0.3m | 条件: ✅building polygons

![img](https://spacenet.ai/wp-content/uploads/2018/10/SpaceNet_Buildings2-300x300.jpg)

SpaceNet 3 — 模态: RGB/NIR/PAN/MS 650²@0.3m | 条件: ✅road centerline(7类)

![img](https://spacenet.ai/wp-content/uploads/2018/10/SpaceNet_Roads-300x300.jpg)

![img](https://storage.googleapis.com/kaggle-datasets-images/1672247/2742916/923aed3fbfbe77620f967f4668ccb382/dataset-cover.png?t=2021-10-30-03-46-15)



------

## 选中数据集详细组成

### SpaceNet 6 MSAW（5模态对齐）

| 数据            | 模态              | 传感器   | GSD  | 波段数 | 原生尺寸 | 备注               |
| --------------- | ----------------- | -------- | ---- | ------ | -------- | ------------------ |
| RGB             | RGB               | —        | 0.5m | 3      | 900²     |                    |
| NIR             | NIR               | —        | 0.5m | 1      | 900²     |                    |
| PAN             | PAN               | —        | 0.5m | 1      | 900²     |                    |
| MS              | MS                | —        | 2.0m | 8      | 225²     |                    |
| SAR             | SAR               | S1       | 0.5m | 1      | 900²     |                    |
| **✅选用**       | building polygons | 人工标注 | —    | —      | —        | **ControlNet条件** |
| building height | 回归值            | 人工标注 | —    | —      | —        | →VLM上下文         |

**处理步骤**：

1. **模态筛选**：保留 RGB、NIR、PAN、MS、SAR + building polygons；building height 保留为VLM上下文
2. **格式读取**：影像为 GeoTIFF，用 `tifffile.imread()` 读取，确认尺寸 900²（RGB/NIR/PAN/SAR）或 225²（MS）；building polygons 为 GeoJSON，用 `geopandas.read_file()` 读取
3. **云筛选**：无需筛选，数据集已精心筛选

原生尺寸：SAR/PAN/RGB/NIR: 900²@0.5m, MS: 225²@2.0m

| 模态        | 原生尺寸  | 操作     | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | --------- | -------- | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 900²@0.5m | 直接裁剪 | 256       | 128    | 5×5=25       | —         |
| PAN         | 900²@0.5m | 直接裁剪 | 512       | 256    | 2×2=4        | —         |
| MS          | 225²@2.0m | 直接裁剪 | 128       | 64     | 2×2=4        | —         |
| segmap      | 900²@0.5m | 直接裁剪 | 256       | 128    | 5×5=25       | 同RGB位置 |

**对齐策略**：PAN的4个512² patch覆盖范围 = MS的4个128² patch覆盖范围 = RGB/NIR/SAR中对应4个256² patch覆盖范围。取PAN/MS的4个位置，对应RGB/NIR/SAR裁出4个256² patch → 每tile**4组对齐patch**。

**覆盖范围对应**：PAN 512²@0.5m = 256m地面，MS 128²@2.0m = 256m地面，RGB 256²@0.5m = 128m地面 → PAN和MS覆盖256m²，RGB覆盖128m²，**不等**。需调整：取PAN中心128m²区域对应RGB的256² patch，或取RGB的256² patch对应PAN中心256²区域。实际取PAN 512² patch的**中心256m**对应RGB 256² patch。

------

### SatlasPretrain（第一档⭐1）

| 数据                        | 模态     | 传感器         | GSD  | 波段数 | 原生尺寸 | 备注                      |
| --------------------------- | -------- | -------------- | ---- | ------ | -------- | ------------------------- |
| RGB                         | RGB      | S2+NAIP        | 10m  | 3      | 512²     |                           |
| NIR                         | NIR      | S2+NAIP        | 10m  | 1      | 512²     |                           |
| MS                          | MS       | S2             | 10m  | 6      | 512²     | B5/B6/B7/B8A/B11/B12      |
| SAR                         | SAR      | S1             | 10m  | 2      | 512²     | VV/VH                     |
| **✅选用**                   | segmap   | 137类7标注类型 | 10m  | 1      | 512²     | **ControlNet条件**; Z+G+C |
| point/polygon/polyline      | 矢量标注 | 人工+GIS+众包  | —    | —      | —        | →VLM上下文                |
| class/regression/properties | 属性标注 | 人工+GIS+众包  | —    | —      | —        | →VLM上下文                |

**处理步骤**：

1. **模态筛选**：保留 S2 光学（12ch）+ S1 SAR（2ch）+ segmap（137类）；point/polygon/polyline 矢量标注、class/regression/properties 属性标注保留为VLM上下文
2. **格式读取**：图像为 .npy，用 `numpy.load()` 读取，确认 512²，14 通道；segmap 用 `numpy.load()` 读取，确认值 0-136
3. **云筛选**：无需筛选，已预处理

原生尺寸：512²@10m

| 模态        | 原生尺寸 | 操作     | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | -------- | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 512²@10m | 直接裁剪 | 256       | 128    | 3×3=9        | —         |
| MS          | 512²@10m | 直接裁剪 | 128       | 64     | 7×7=49       | —         |
| segmap      | 512²@10m | 直接裁剪 | 256       | 128    | 3×3=9        | 同RGB位置 |

**对齐策略**：RGB裁3×3=9个256² patch，MS裁7×7=49个128² patch。取RGB的9个patch中心位置，对应MS中相同地面中心的128² patch → 每tile**9组对齐patch**。

------

### M3LEO（第一档⭐2）

| 数据           | 模态              | 传感器      | GSD  | 波段数 | 存储尺寸 | 备注                             |
| -------------- | ----------------- | ----------- | ---- | ------ | -------- | -------------------------------- |
| s1grd          | SAR               | S1 GRD      | 10m  | 16     | 448²     | VV/VH×4季×2轨道                  |
| s2srm          | MS                | S2 L2A      | 10m  | 10     | 448²     | B2/B3/B4/B5/B6/B7/B8/B8A/B11/B12 |
| **✅选用**      | WorldCover segmap | ESA         | 10m  | 1      | 448²     | **ControlNet条件**; 11类         |
| srtmdem        | DEM               | SRTM        | 30m  | 2      | 448²     | →VLM上下文                       |
| modis44b006veg | 植被              | MODIS VCF   | 250m | 5      | 448²     | →VLM上下文                       |
| biomass        | 生物量            | ESA CCI     | 90m  | 1      | 45×55⚠️   | →VLM上下文; 粗分辨率             |
| ghsbuilts      | 建成区            | EC JRC GHSL | 100m | 1      | 75×56⚠️   | →VLM上下文; 粗分辨率             |

**M3LEO关键数据**：6区域，~17M tiles，18.73TB，2018-2020年，448²@10m。每个tile覆盖4480m×4480m。60%训练/20%验证/20%测试。S1/S2/LC/DEM/veg精细对齐448²，biomass/built-up粗分辨率存储需上采样。

**处理步骤**：

1. **模态筛选**：保留 s1grd（SAR 16ch）、s2srm（S2 10ch）、esaworldcover（segmap 11类）、srtmdem（DEM 2ch）、modis44b006veg（植被5ch）；biomass（90m）、ghsbuilts（100m）、gssic、gunw（粗分辨率）保留为VLM上下文
2. **格式读取**：Parquet 格式，原始 .tif/.nc 文件以二进制存储在行中，需解压后用 `numpy.load()` 读取，确认 448²
3. **云筛选**：无需筛选，已预处理（S2 为云掩膜月度中值合成）

原生尺寸：448²@10m（GEE重采样后统一448²）

| 模态              | 原生尺寸 | 操作               | crop_size | stride | patches/tile | 对齐方式  |
| ----------------- | -------- | ------------------ | --------- | ------ | ------------ | --------- |
| s1grd(SAR)        | 448²@10m | 直接裁剪           | 256       | 128    | 2×2=4        | —         |
| s2srm(RGB/NIR/MS) | 448²@10m | 直接裁剪           | 256/128   | 128/64 | 4/36         | —         |
| WorldCover segmap | 448²@10m | 直接裁剪           | 256       | 128    | 2×2=4        | 同SAR位置 |
| srtmdem(DEM)      | 448²@10m | 直接裁剪           | 256       | 128    | 2×2=4        | 同SAR位置 |
| modis(veg)        | 448²@10m | 直接裁剪           | 128       | 64     | 6×6=36       | 同MS位置  |
| biomass           | 45×55    | 上采样至448²后裁剪 | 256       | 128    | 2×2=4        | 同SAR位置 |
| ghsbuilts         | 75×56    | 上采样至448²后裁剪 | 256       | 128    | 2×2=4        | 同SAR位置 |

**对齐策略**：SAR裁2×2=4个256² patch，MS裁6×6=36个128² patch。取SAR的4个patch中心位置，对应MS中相同地面中心的128² patch → 每tile**4组对齐patch**。

------

### CloudLULC-Set（第一档⭐3，需筛选无云/少云）

| 数据           | 模态   | 传感器                | GSD  | 波段数 | 原生尺寸 | 备注                                                  |
| -------------- | ------ | --------------------- | ---- | ------ | -------- | ----------------------------------------------------- |
| RGB            | RGB    | S2 L2A                | 10m  | 3      | 512²     |                                                       |
| NIR            | NIR    | S2 L2A                | 10m  | 1      | 512²     |                                                       |
| MS             | MS     | S2 L2A                | 10m  | 9      | 512²     | B1/B5/B6/B7/B8/B8A/B9/B11/B12                         |
| SAR            | SAR    | S1 GRD IW             | 10m  | 2      | 512²     | VV/VH                                                 |
| **✅选用**      | segmap | 专家判读+辅助交叉验证 | 10m  | 1      | 512²     | **ControlNet条件**; 6类                               |
| cloud_coverage | 云覆盖 | Fmask 4.6             | —    | —      | —        | →VLM上下文; 5区间(0-20%/20-40%/40-60%/60-80%/80-100%) |

**CloudLULC-Set关键数据**：40,223 samples，48 ROIs，578,690 km²，512²@10m，全球分布。

**6类LULC分布**：

| 类别 | Forest | Grassland | Cropland | Urban/Built-up | Barren | Water  |
| ---- | ------ | --------- | -------- | -------------- | ------ | ------ |
| 占比 | 29.60% | 28.53%    | 15.98%   | 2.52%          | 8.41%  | 14.94% |

**时间一致性**：云污染光学影像与参考光学影像平均时间间隔5.17天，光学与SAR平均时间偏移3.35天

**处理步骤**：

1. **模态筛选**：保留全部模态（RGB、NIR、MS、SAR、segmap、cloud_coverage）

2. **格式读取**：图像为 .npy，用 `numpy.load()` 读取，确认 512²；segmap 确认值 0-5（6类）

3. **✅ 云筛选（必须执行）**：读取 `cloud_coverage` 元数据，**仅保留 0-20% 区间的样本**（云覆盖≤20%）。我们需可用的 RGB/NIR 光学图像，云覆盖严重的样本无法使用。预期保留约 20-30%。代码：

   ```python
   filtered = [s for s in samples if s['cloud_coverage'] == '0-20%']
   ```

4. **类别不平衡**：Urban/Built-up 仅占 2.52%，训练时可考虑类别加权

原生尺寸：512²@10m

| 模态        | 原生尺寸 | 操作     | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | -------- | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 512²@10m | 直接裁剪 | 256       | 128    | 3×3=9        | —         |
| MS          | 512²@10m | 直接裁剪 | 128       | 64     | 7×7=49       | —         |
| segmap      | 512²@10m | 直接裁剪 | 256       | 128    | 3×3=9        | 同RGB位置 |

**对齐策略**：同SatlasPretrain，取RGB 9个patch中心对应MS 9个128² patch → 每tile**9组对齐patch**。

------

### TerraMesh（第一档4）

| 数据        | 模态        | 传感器           | GSD  | 波段数 | 原生尺寸 | 备注                     |
| ----------- | ----------- | ---------------- | ---- | ------ | -------- | ------------------------ |
| S2L2A       | RGB+NIR+MS  | S2 L2A           | 10m  | 12     | 264²     | B1-B12                   |
| S2RGB       | RGB         | S2               | 10m  | 3      | 264²     | 仅RGB                    |
| S1GRD/S1RTC | SAR         | S1               | 10m  | 2      | 264²     | VV/VH，每个样本仅含一种  |
| DEM         | DEM         | Copernicus DEM   | 10m  | 1      | 264²     | →VLM上下文               |
| NDVI        | NDVI        | 计算值           | 10m  | 1      | 264²     | →VLM上下文               |
| **✅选用**   | LULC segmap | ESRI全球土地覆盖 | 10m  | 1      | 264²     | **ControlNet条件**; 10类 |
| cloud_mask  | 云掩膜      | S2元数据         | 10m  | 1      | 264²     | →VLM上下文; S2和LULC提供 |

**TerraMesh关键数据**：9M+ co-registered patches，train/val splits，889 shard文件/shard最多10240样本。数据来源：MajorTom-Core + SSL4EO-S12。85%的S1/S2时间戳在一周内对齐，1%时间差超过1年。

**TerraMesh-Masks（条件补充，非独立数据集）**：另有配套的 TerraMesh-Masks（二值水体mask + 图像caption），可并入TerraMesh条件体系使用（mask→额外segmap条件、caption→text prompt），不单独入库。

**处理步骤**：

1. **模态筛选**：保留 S2L2A（12ch）、S1GRD 或 S1RTC（SAR 2ch，每个样本仅含一种）、LULC（segmap 10类）、DEM、NDVI、cloud_mask；S2L1C（未做大气校正）、S2RGB（L2A已包含）保留为VLM上下文

2. **格式读取**：Zarr Zip 格式（.zarr.zip），用 `xarray.load_zarr()` 读取，确认 264²：

   ```python
   import xarray as xr
   ds = xr.load_zarr('sample.zarr.zip')
   ```

3. **云筛选**：无需筛选，已预处理。每个样本仅含 S1GRD 或 S1RTC 之一

4. **标签检查**：LULC segmap 确认值 0-9（10类）

5. **元数据**：train_metadata.parquet / val_metadata.parquet 包含样本概览

原生尺寸：264²@10m

| 模态        | 原生尺寸 | 操作                              | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | --------------------------------- | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 264²@10m | 直接裁剪（1个patch，边缘padding） | 256       | —      | 1            | —         |
| MS          | 264²@10m | 直接裁剪（1个patch，边缘padding） | 128       | —      | 1            | —         |
| ESRI segmap | 264²@10m | 直接裁剪                          | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：原生264²仅能裁1个256² patch（边缘8像素padding），MS裁1个128² patch → 每tile**1组对齐patch**。

------

### SEN12MS（第一档5）

| 数据      | 模态         | 传感器        | GSD  | 波段数 | 原生尺寸 | 备注                                |
| --------- | ------------ | ------------- | ---- | ------ | -------- | ----------------------------------- |
| RGB       | RGB          | S2            | 10m  | 3      | 256²     |                                     |
| NIR       | NIR          | S2            | 10m  | 1      | 256²     |                                     |
| MS        | MS           | S2            | 10m  | 8      | 256²     | B5/B6/B7/B8A/B11/B12+大气2波段      |
| SAR       | SAR          | S1            | 10m  | 2      | 256²     | VV/VH                               |
| **✅选用** | MODIS segmap | MODIS MCD12Q1 | 500m | 1      | 256²     | **ControlNet条件**; 33类; ⚠️质量较低 |
| metadata  | 季节/位置CSV | 人工标注      | —    | —      | —        | →VLM上下文                          |

**处理步骤**：

1. **模态筛选**：保留全部模态（S2 15ch、S1 2ch、MODIS segmap 33类）
2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取，确认 256²，S2 15ch + S1 2ch
3. **云筛选**：无需筛选，已预处理
4. **标签注意**：MODIS segmap 分辨率 500m，远低于影像 10m，标签质量较低
5. **四季结构**：按4种子分为4季（ROIs1158/ROIs1868/ROIs1970/ROIs2017），文件命名 `ROIs{种子}{季节}{数据类型}p{patchID}.tif`；seasons.csv 提供实际季节分类，可用于 train/test 划分

原生尺寸：256²@10m

| 模态         | 原生尺寸 | 操作               | crop_size | stride | patches/tile | 对齐方式  |
| ------------ | -------- | ------------------ | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR  | 256²@10m | 直接裁剪（无裁剪） | 256       | —      | 1            | —         |
| MS           | 256²@10m | 直接裁剪           | 128       | 64     | 2×2=4        | —         |
| MODIS segmap | 256²@10m | 直接裁剪           | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：RGB/NIR/SAR原生256²无需裁剪，MS裁2×2=4个128² patch。取RGB中心位置对应MS的1个128² patch → 每tile**1组对齐patch**。

------

### OlmoEarth（第一档6）

| 数据                           | 模态              | 传感器               | GSD               | 波段数 | 原生尺寸  | 备注                                                 |
| ------------------------------ | ----------------- | -------------------- | ----------------- | ------ | --------- | ---------------------------------------------------- |
| sentinel2_l2a_monthly(10m文件) | RGB+NIR           | S2 L2A               | 10m               | 4      | 256²      | B02/B03/B04/B08，月度镶嵌×12期(360天，channel轴堆叠) |
| sentinel2_l2a_monthly(20m文件) | MS                | S2 L2A               | 20m               | 6      | 128²      | B05/B06/B07/B8A/B11/B12                              |
| sentinel2_l2a_monthly(40m文件) | MS                | S2 L2A               | 40m               | 2      | 64²       | B01/B09(60m重采样至40m) →VLM上下文                   |
| sentinel1_monthly              | SAR               | S1 IW GRD            | 10m               | 2      | 256²      | VV/VH，月度×12期，MPC RTC处理                        |
| **✅选用**                      | WorldCover segmap | ESA WorldCover 2021  | 10m               | 1      | 256²      | **ControlNet条件**; 11类(10-100)                     |
| cdl                            | CDL作物类型       | USDA CDL(仅美国本土) | 10m               | 1      | 256²      | ~100类作物，最近年份 →第二条件(仅美国)               |
| landsat_monthly                | 光学(非目标)      | L8/L9 OLI-TIRS       | 10m(B8)/20m(其余) | 11     | 256²/128² | 15m/30m重采样；B8非真PAN(比RGB粗) →VLM上下文         |
| srtm                           | DEM               | SRTM                 | 10m               | 1      | 256²      | →VLM上下文                                           |
| era5_monthly                   | 气候(6变量×12期)  | ECMWF ERA5           | 2560m             | 72     | 1×1       | →VLM上下文                                           |
| openstreetmap_raster           | 矢量栅格化        | OSM                  | 2.5m              | —      | 1024²     | →VLM上下文                                           |
| worldcereal                    | 作物地图          | ESA WorldCereal 2021 | 10m               | —      | 256²      | →VLM上下文                                           |
| wri_canopy_height              | 冠层高度          | WRI/Meta             | 10m               | 1      | 256²      | →VLM上下文                                           |

**OlmoEarth关键数据**：285,288 samples；每个sample = 2560m×2560m UTM网格 + 360天时间范围，S1/S2/Landsat各最多12期30天镶嵌（缺数据时部分模态/时相缺失，需按前缀过滤）。采样基于OSM约120类要素（每类≤10K格网），时间范围2016.01-2024.12（美国本土75%以NAIP影像为中心）。全量16.15TB（50GB tar/模态）；按需(S2+S1+WorldCover+CDL)≈4.6TB。CC-BY-4.0。论文: arXiv 2511.13655。

**处理步骤**：

1. **模态筛选**：保留 S2 10m+20m文件（RGB+NIR+6波段MS）、S1 vv/vh、WorldCover、CDL(仅美国)；S2 40m(B01/B09)、Landsat、SRTM/ERA5/OSM/WorldCereal/冠层高度 → VLM上下文
2. **格式读取**：GeoTIFF（tar解包），同一sample通过文件名前缀（如`EPSG:32610_150_-2089`）跨模态文件夹配对；多时相已在channel轴堆叠
3. **云筛选**：月度镶嵌已按质量合成，仍建议抽查云残留；缺模态样本按前缀过滤
4. **标签检查**：WorldCover值域10,20,...,100（11类）；CDL为作物类型码（仅美国本土样本有）
5. **时相扩充**：12期月度镶嵌可作为独立样本扩充（每期1组patch，最多×12→~3.4M）或仅取1期

原生尺寸：256²@10m（RGB/NIR/SAR）、128²@20m（MS）

| 模态              | 原生尺寸 | 操作           | crop_size | stride | patches/tile | 对齐方式  |
| ----------------- | -------- | -------------- | --------- | ------ | ------------ | --------- |
| RGB/NIR           | 256²@10m | 直接（无裁剪） | 256       | —      | 1            | —         |
| MS                | 128²@20m | 直接（无裁剪） | 128       | —      | 1            | —         |
| SAR               | 256²@10m | 直接（无裁剪） | 256       | —      | 1            | —         |
| WorldCover segmap | 256²@10m | 直接           | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：**原生即目标尺寸，零裁剪零插值**——RGB/NIR/SAR 256²@10m与MS 128²@20m同覆盖2560m×2560m（GSD_MS=2×GSD_RGB完全吻合），S1/S2/WorldCover/CDL同UTM网格像素级对应。

------

### MADS（第一档7）

| 数据        | 模态      | 传感器     | GSD    | 波段数 | 原生尺寸 | 备注                                                         |
| ----------- | --------- | ---------- | ------ | ------ | -------- | ------------------------------------------------------------ |
| SAR         | SAR       | S1 GRD IW  | 10m    | 2      | 1371×888 | VV/VH                                                        |
| MS          | MS        | S2 L2A     | 10m    | 12     | 1371×888 | B1-B12                                                       |
| HSI         | HSI       | HySpex机载 | 2.2m   | 416    | —        | VNIR(160ch)+SWIR(256ch)                                      |
| DSM         | DSM       | DLR 3K     | 0.25m  | 1      | —        | 重采样至0.3m                                                 |
| GIS         | GIS       | OSM        | —      | —      | —        | 建筑/土地利用/水体层                                         |
| **✅选用**   | segmap    | 人工标注   | 0.2m   | 1      | 3子区域  | **ControlNet条件**; 6类(pavement/low vegetation/soil/tree/roof/water) |
| 端元光谱    | 光谱库    | 人工标注   | —      | 859条  | —        | →VLM上下文                                                   |
| S2eteS_S2   | MS(模拟)  | 模拟S2     | 10m    | 4      | —        | →VLM上下文                                                   |
| EeteS_EnMAP | HSI(模拟) | 模拟EnMAP  | 10/30m | 242    | —        | →VLM上下文                                                   |

**处理步骤**：

1. **模态筛选**：保留 SAR（S1 GRD 2ch）、MS（S2 L2A 12ch）、DSM、segmap（6类LC，仅3子区域）；HSI（416波段）、GIS（OSM）、端元光谱（859条）、模拟产品保留为VLM上下文
2. **格式读取**：S1/S2/DSM 为 GeoTIFF，用 `tifffile.imread()` 读取，确认 S1/S2 尺寸 1371×888@10m
3. **云筛选**：无需筛选，已预处理
4. **子区域限制**：segmap 仅覆盖 3 个子区域（Augsburg等），处理时需检查每个 patch 是否有对应 segmap，无 segmap 的 patch 仅保留影像用于 VLM 上下文

原生尺寸：S1/S2: 1371×888@10m

| 模态         | 原生尺寸     | 操作              | crop_size | stride | patches/tile | 对齐方式  |
| ------------ | ------------ | ----------------- | --------- | ------ | ------------ | --------- |
| SAR(RGB/NIR) | 1371×888@10m | 直接裁剪          | 256       | 128    | 9×5=45       | —         |
| MS           | 1371×888@10m | 直接裁剪          | 128       | 64     | 21×12=252    | —         |
| segmap       | 3子区域@0.2m | 降采样至10m后裁剪 | 256       | 128    | 按子区域大小 | 同RGB位置 |

**对齐策略**：SAR裁9×5=45个256² patch，MS裁21×12=252个128² patch。取SAR的45个patch中心位置，对应MS中相同地面中心的128² patch → 每tile约**45组对齐patch**（segmap仅覆盖3个子区域，部分patch无segmap）。

------

### LuojiaSET-OSFCR（第一档8，需筛选无云/少云）

| 数据        | 模态   | 传感器 | GSD  | 波段数 | 原生尺寸 | 备注                      |
| ----------- | ------ | ------ | ---- | ------ | -------- | ------------------------- |
| RGB         | RGB    | S2     | 10m  | 3      | 256²     |                           |
| NIR         | NIR    | S2     | 10m  | 1      | 256²     |                           |
| MS          | MS     | S2     | 10m  | 6      | 256²     |                           |
| SAR         | SAR    | S1     | 10m  | 2      | 256²     | VV/VH                     |
| **✅选用**   | segmap | LC分类 | 10m  | 1      | 256²     | **ControlNet条件**; P(QC) |
| cloud_cover | 云覆盖 | 元数据 | —    | —      | —        | →VLM上下文; 0-80%         |

**处理步骤**：

1. **模态筛选**：保留全部模态（RGB、NIR、MS、SAR、segmap、cloud_cover）

2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取，确认 256²

3. **✅ 云筛选（必须执行）**：读取 `cloud_cover` 元数据，**仅保留 ≤20% 的样本**。我们需可用的光学图像，云覆盖严重样本无法使用。预期保留约 40-60%。代码：

   ```python
   filtered = [s for s in samples if s['cloud_cover'] <= 20]
   ```

4. **标签检查**：segmap 确认值 0-N（LC 分类数）

原生尺寸：256²@10m

| 模态        | 原生尺寸 | 操作               | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | ------------------ | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 256²@10m | 直接裁剪（无裁剪） | 256       | —      | 1            | —         |
| MS          | 256²@10m | 直接裁剪           | 128       | 64     | 2×2=4        | —         |
| segmap      | 256²@10m | 直接裁剪           | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：同SEN12MS，每tile **1组对齐patch**。

------

### HunanMultimodal（第一档9）

| 数据      | 模态   | 传感器   | GSD  | 波段数 | 原生尺寸 | 备注                        |
| --------- | ------ | -------- | ---- | ------ | -------- | --------------------------- |
| RGB       | RGB    | S2       | 10m  | 3      | 256²     |                             |
| NIR       | NIR    | S2       | 10m  | 1      | 256²     |                             |
| MS        | MS     | S2       | 10m  | 6      | 256²     |                             |
| SAR       | SAR    | S1       | 10m  | 2      | 256²     | VV/VH                       |
| **✅选用** | segmap | 人工标注 | 10m  | 1      | 256²     | **ControlNet条件**; 7类LULC |
| DEM       | DEM    | —        | 10m  | 1      | 256²     | →VLM上下文                  |

**处理步骤**：

1. **模态筛选**：保留全部模态（RGB、NIR、MS、SAR、segmap、DEM）
2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取，确认 256²
3. **云筛选**：无需筛选，已预处理
4. **标签检查**：segmap 确认值 0-6（7类 LULC）

原生尺寸：256²@10m

| 模态        | 原生尺寸 | 操作               | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | ------------------ | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 256²@10m | 直接裁剪（无裁剪） | 256       | —      | 1            | —         |
| MS          | 256²@10m | 直接裁剪           | 128       | 64     | 2×2=4        | —         |
| segmap      | 256²@10m | 直接裁剪           | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：同SEN12MS，每tile **1组对齐patch**。

------

### DFC20/DFC2020（第一档10）

| 数据      | 模态   | 传感器            | GSD  | 波段数 | 原生尺寸 | 备注                         |
| --------- | ------ | ----------------- | ---- | ------ | -------- | ---------------------------- |
| RGB       | RGB    | S2                | 10m  | 3      | 256²     |                              |
| NIR       | NIR    | S2                | 10m  | 1      | 256²     |                              |
| MS        | MS     | S2                | 10m  | 8      | 256²     |                              |
| SAR       | SAR    | S1                | 10m  | 2      | 256²     | VV/VH                        |
| **✅选用** | segmap | MODIS MCD12Q1叠加 | 10m  | 1      | 256²     | **ControlNet条件**; 10类LULC |

**处理步骤**：

1. **模态筛选**：保留全部模态（S2 13ch、S1 2ch、segmap 10类）
2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取，确认 256²
3. **云筛选**：无需筛选，已预处理
4. **标签检查**：segmap 确认值 0-9（10类 LULC）

原生尺寸：256²@10m

| 模态        | 原生尺寸 | 操作               | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | ------------------ | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 256²@10m | 直接裁剪（无裁剪） | 256       | —      | 1            | —         |
| MS          | 256²@10m | 直接裁剪           | 128       | 64     | 2×2=4        | —         |
| segmap      | 256²@10m | 直接裁剪           | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：同SEN12MS，每tile **1组对齐patch**。

------

### MultiSenGE（第一档11）

| 数据      | 模态       | 传感器             | GSD  | 波段数 | 原生尺寸 | 备注                                         |
| --------- | ---------- | ------------------ | ---- | ------ | -------- | -------------------------------------------- |
| S2        | RGB+NIR+MS | S2                 | 10m  | 12     | 256²     | 多时相                                       |
| S1        | SAR        | S1                 | 10m  | 2      | 256²     | VV/VH，多时相                                |
| **✅选用** | segmap     | OCSGE2-GEOGRANDEST | 10m  | 1      | 256²     | **ControlNet条件**; 10类LULC（原14类合并后） |

**MultiSenGE关键数据**：8,157 multi-temporal patches，Grand-Est region，France，OCSGE2-GEOGRANDEST ground reference。

**类别合并说明**（原14类→10类）：

- Orchards(8) 合并入 Vineyards(7)
- Groves, Hedges(9) 合并入 Forests(10)
- Open Spaces, Mineral(12) 合并入 Forests(10)
- Wetlands(13) 合并入 Water Surfaces(14)

**处理步骤**：

1. **模态筛选**：保留全部模态（S2 12ch 多时相、S1 2ch 多时相、segmap 10类）
2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取，确认 256²，已重投影至 Lambert-93 (EPSG:2154)
3. **云筛选**：无需筛选，已预处理
4. **标签检查**：segmap 确认值 0-9（10类 LULC，原14类合并后）
5. **多时相**：S1/S2 均为多时相数据，每个 patch 包含多个时间点观测

原生尺寸：256²@10m

| 模态        | 原生尺寸 | 操作               | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | ------------------ | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 256²@10m | 直接裁剪（无裁剪） | 256       | —      | 1            | —         |
| MS          | 256²@10m | 直接裁剪           | 128       | 64     | 2×2=4        | —         |
| segmap      | 256²@10m | 直接裁剪           | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：同SEN12MS，每tile **1组对齐patch**。

------

### MMEarth（第二档1）

| 数据             | 模态              | 传感器    | GSD  | 波段数 | 原生尺寸 | 备注                     |
| ---------------- | ----------------- | --------- | ---- | ------ | -------- | ------------------------ |
| S2               | RGB+NIR+MS        | S2        | 10m  | 12     | 128²     | B1-B12                   |
| S1               | SAR               | S1        | 10m  | 2      | 128²     | VV/VH                    |
| **✅选用**        | WorldCover segmap | ESA       | 10m  | 1      | 128²     | **ControlNet条件**; 11类 |
| DEM              | DEM               | ASTER DEM | 30m  | 1      | 128²     | →VLM上下文               |
| Dynamic World    | 土地覆盖          | DW        | 10m  | 1      | 128²     | →VLM上下文               |
| Canopy Height    | 冠层高度          | —         | —    | 1      | 128²     | →VLM上下文               |
| Biome            | 生物群落          | —         | —    | 1      | 128²     | →VLM上下文; 14 biomes    |
| Ecoregion        | 生态区            | —         | —    | 1      | 128²     | →VLM上下文               |
| ERA5 temp/precip | 气象              | ERA5      | —    | 2      | 128²     | →VLM上下文               |
| Geolocation      | 坐标              | —         | —    | 2      | 128²     | →VLM上下文               |
| S2 date          | 观测日期          | —         | —    | 1      | 128²     | →VLM上下文               |

**MMEarth关键数据**：1.2M locations，12 aligned modalities（6 pixel-level + 6 image-level），14 biomes，2017-2020年。128²@10m覆盖1.28km×1.28km。

**两个可用版本**：

| 版本              | 尺寸 | 数据量 | 大小  |
| ----------------- | ---- | ------ | ----- |
| MMEarth（完整版） | 128² | 1.2M   | 597GB |
| MMEarth100k       | 128² | 100k   | 48GB  |

**处理步骤**：

1. **模态筛选**：保留全部6种像素级模态（S2 12ch、S1 2ch、WorldCover segmap 11类、DEM、Dynamic World、Canopy Height）+ 6种图像级模态（Biome、Ecoregion、ERA5 temp/precip、Geolocation、S2 date）
2. **格式读取**：.npy，用 `numpy.load()` 读取，确认 128²
3. **云筛选**：无需筛选，已预处理
4. **标签检查**：WorldCover segmap 确认值 0-10（11类）
5. **插值**：RGB/NIR/SAR 需 2× 双线性插值至 256²，MS 直接用原生 128²，segmap 用最近邻插值
6. **版本**：建议完整版（128²，597GB），MMEarth100k 用于快速验证

原生尺寸：128²@10m

| 模态              | 原生尺寸 | 操作                  | crop_size | stride | patches/tile | 对齐方式  |
| ----------------- | -------- | --------------------- | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR       | 128²@10m | **2×双线性插值**→256² | 256       | —      | 1            | —         |
| MS                | 128²@10m | 直接裁剪（无裁剪）    | 128       | —      | 1            | —         |
| WorldCover segmap | 128²@10m | **最近邻插值**→256²   | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：RGB/NIR/SAR插值到256²后覆盖1280m×1280m，MS原生128²也覆盖1280m×1280m → 每tile **1组对齐patch**。1个MS像素 = 插值RGB的2×2像素。

------

### BigEarthNet（第二档2）

| 数据      | 模态   | 传感器         | GSD  | 波段数 | 原生尺寸 | 备注                                         |
| --------- | ------ | -------------- | ---- | ------ | -------- | -------------------------------------------- |
| RGB       | RGB    | S2 L2A         | 10m  | 3      | 120²     | B02/B03/B04                                  |
| NIR       | NIR    | S2 L2A         | 10m  | 1      | 120²     | B08                                          |
| MS        | MS     | S2 L2A         | 10m  | 8      | 120²     | B01/B05/B06/B07/B8A/B09/B11/B12              |
| SAR       | SAR    | S1 GRD         | 10m  | 2      | 128²     | VV/VH                                        |
| **✅选用** | segmap | CORINE CLC2018 | 10m  | 1      | 120²     | **ControlNet条件**; 19类多标签，像素级参考图 |
| metadata  | 元数据 | GIS+规则       | —    | —      | —        | →VLM上下文                                   |

**BigEarthNet关键数据**：549,488 patches，115 Sentinel-2 tiles(2017.06-2018.05)，10个欧洲国家(Austria/Belgium/Finland/Ireland/Kosovo/Lithuania/Luxembourg/Portugal/Serbia/Switzerland)，CLC2018 v2020_u1标签

**处理步骤**：

1. **模态筛选**：保留全部模态（S2 12ch、S1 2ch、segmap 19类多标签 + 像素级参考图）
2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取；S2 120²，S1 128²
3. **云筛选**：无需筛选，已预处理
4. **标签检查**：segmap 确认值 0-18（19类 CORINE CLC2018），多标签
5. **尺寸对齐**：S1 128² 裁剪至 120²（与 S2 对齐），再分别插值至 256²

原生尺寸：S1=128²@10m, S2=120²@10m

| 模态       | 原生尺寸 | 操作                     | crop_size | stride | patches/tile | 对齐方式  |
| ---------- | -------- | ------------------------ | --------- | ------ | ------------ | --------- |
| RGB/NIR/MS | 120²@10m | **2.13×双线性插值**→256² | 256       | —      | 1            | —         |
| SAR        | 128²@10m | **2×双线性插值**→256²    | 256       | —      | 1            | —         |
| segmap     | 120²@10m | **最近邻插值**→256²      | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：S2(120²)和S1(128²)覆盖范围不同（1200m vs 1280m），需先裁剪S1至120²@10m（1200m覆盖），再分别插值到256² → 每tile **1组对齐patch**。

------

### CerraData-4MM（第二档3）

| 数据      | 模态      | 传感器                 | GSD  | 波段数 | 原生尺寸 | 备注                                      |
| --------- | --------- | ---------------------- | ---- | ------ | -------- | ----------------------------------------- |
| RGB       | RGB       | S2                     | 10m  | 3      | 128²     | B02/B03/B04                               |
| NIR       | NIR       | S2                     | 10m  | 1      | 128²     | B08                                       |
| MS        | MS        | S2                     | 10m  | 8      | 128²     | B01/B05/B06/B07/B8A/B09/B11/B12           |
| SAR       | SAR       | S1                     | 10m  | 2      | 128²     | VV/VH                                     |
| **✅选用** | segmap L2 | TerraClass DL+专家验证 | 10m  | 1      | 128²     | **ControlNet条件**; 14子类                |
| **✅选用** | segmap L1 | 同上                   | 10m  | 1      | 128²     | **ControlNet条件**; 7大类（L2的上级分类） |

**CerraData-4MM类别详情**（Bico do Papagaio生态区，TerraClass 2022）：

| L1类别      | L1 ID | L2类别                               | L2 ID | RGB颜色     |
| ----------- | ----- | ------------------------------------ | ----- | ----------- |
| Pasture     | 0     | Pasture (Pa)                         | 0     | 206,239,98  |
| Forest      | 1     | Primary Natural Vegetation (V1)      | 1     | 22,152,13   |
| Agriculture | 2     | Secondary Natural Vegetation (V2)    | 2     | 31,212,18   |
| Mining      | 3     | Mining (Mg)                          | 4     | 176,176,176 |
| Building    | 4     | Urban area (UA)                      | 5     | 223,124,38  |
| Water body  | 5     | Water body (Wt)                      | 3     | 19,50,255   |
| Other Uses  | 6     | Other Built area (OB)                | 6     | 250,128,114 |
| —           | —     | Forestry (Ft)                        | 7     | 85,107,47   |
| —           | —     | Perennial Agriculture (PR)           | 8     | 230,32,108  |
| —           | —     | Semi-perennial Agriculture (SP)      | 9     | 139,105,20  |
| —           | —     | Temporary agriculture 1 cycle (T1)   | 10    | 255,215,0   |
| —           | —     | Temporary agriculture 1+ cycle (T1+) | 11    | 255,255,0   |
| —           | —     | Other Uses (OU)                      | 12    | 117,10,194  |
| —           | —     | Deforestation 2022 (Df)              | 13    | 205,0,0     |

**数据量**：30,322 patches/modality，128²@10m，2022年采集

**处理步骤**：

1. **模态筛选**：保留全部模态（S2 12ch、S1 2ch、segmap L2 14子类 + L1 7大类）
2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取，确认 128²
3. **云筛选**：无需筛选，已预处理
4. **标签检查**：segmap L2 确认值 0-13（14子类），segmap L1 确认值 0-6（7大类）
5. **插值**：RGB/NIR/SAR 需 2× 双线性插值至 256²，MS 直接用原生 128²，segmap L1/L2 用最近邻插值

原生尺寸：128²@10m

| 模态        | 原生尺寸 | 操作                  | crop_size | stride | patches/tile | 对齐方式  |
| ----------- | -------- | --------------------- | --------- | ------ | ------------ | --------- |
| RGB/NIR/SAR | 128²@10m | **2×双线性插值**→256² | 256       | —      | 1            | —         |
| MS          | 128²@10m | 直接裁剪（无裁剪）    | 128       | —      | 1            | —         |
| segmap L2   | 128²@10m | **最近邻插值**→256²   | 256       | —      | 1            | 同RGB位置 |
| segmap L1   | 128²@10m | **最近邻插值**→256²   | 256       | —      | 1            | 同RGB位置 |

**对齐策略**：同MMEarth，每tile **1组对齐patch**。

------

### FLAIR-HUB（PAN补充⭐1）

| 数据      | 模态         | 传感器              | GSD             | 波段数   | 原生尺寸 | 备注                              |
| --------- | ------------ | ------------------- | --------------- | -------- | -------- | --------------------------------- |
| RGBI      | RGB+NIR      | BD ORTHO航空        | 0.2m            | 4(RGBI)  | 512²     | 8bit                              |
| PAN       | PAN          | BD ORTHO HISTORIQUE | ~0.4m           | 1        | 256²     | 1947-1965历史影像，需2×插值至512² |
| SPOT MS   | MS           | SPOT 6-7            | 1.6m(重采样)    | 4(RGBI)  | 64²      | 需2×插值至128²                    |
| S2        | MS           | Sentinel-2          | 10.24m(重采样)  | 10       | —        | 时间序列，→VLM上下文              |
| S1        | SAR          | Sentinel-1          | 10.24m(重采样)  | 2(VV/VH) | —        | 时间序列(升/降轨)，→VLM上下文     |
| **✅选用** | CoSIA segmap | 航空照片判读        | 0.2m            | 1        | 512²     | **ControlNet条件**; 15-19类LC     |
| LPIS      | 作物分类     | CAP申报             | 0.2m            | 1        | 512²     | →VLM上下文; 23类                  |
| DSM/DTM   | 高程         | 3K相机              | DSM 0.2m/DTM 1m | 2        | 512²     | →VLM上下文                        |

**FLAIR-HUB关键数据**：241,100 patches(512²)，2,822 ROIs，2,528 km²，74个法国省，~750GB

**处理步骤**：

1. **模态筛选**：保留 RGBI（4ch@0.2m）、PAN（1ch@0.4m）、SPOT MS（4ch@1.6m）、CoSIA segmap（15-19类LC）；S2/S1 时间序列、LPIS 作物分类、DSM/DTM 保留为VLM上下文
2. **格式读取**：GeoTIFF，用 `tifffile.imread()` 读取；RGBI 512²、PAN 256²、SPOT MS 64²
3. **云筛选**：无需筛选，已预处理
4. **标签检查**：CoSIA segmap 确认值 0-18（15-19类 LC）
5. **插值**：PAN 256²@0.4m 需 2× 双线性插值至 512²，MS 64²@1.6m 需 2× 双线性插值至 128²，segmap 用最近邻插值

原生尺寸：RGB/NIR: 512²@0.2m, PAN: 256²@0.4m, MS: 64²@1.6m

| 模态    | 原生尺寸  | 操作                  | crop_size | stride | patches/tile | 对齐方式  |
| ------- | --------- | --------------------- | --------- | ------ | ------------ | --------- |
| RGB/NIR | 512²@0.2m | 直接裁剪              | 256       | 128    | 3×3=9        | —         |
| PAN     | 256²@0.4m | **2×双线性插值**→512² | 512       | —      | 1            | —         |
| MS      | 64²@1.6m  | **2×双线性插值**→128² | 128       | —      | 1            | —         |
| segmap  | 512²@0.2m | 直接裁剪              | 256       | 128    | 3×3=9        | 同RGB位置 |

**对齐策略**：RGB/NIR裁3×3=9个256² patch，PAN插值到512²后裁1个512² patch，MS插值到128²后裁1个128² patch。PAN 512²@0.4m = 204.8m地面，MS 128²@1.6m = 204.8m地面，RGB 256²@0.2m = 51.2m地面 → PAN和MS覆盖204.8m²，RGB覆盖51.2m²。取PAN/MS的1个patch，对应RGB的中心4个256² patch（102.4m² ≈ 204.8m²的一半）→ 每tile取PAN中心区域对应RGB的**4组对齐patch**。

------

### SpaceNet 2（PAN补充，building polygons）

| 数据      | 模态              | 传感器   | GSD  | 波段数 | 原生尺寸 | 备注               |
| --------- | ----------------- | -------- | ---- | ------ | -------- | ------------------ |
| RGB       | RGB               | WV-2     | 0.3m | 3      | 650²     |                    |
| NIR       | NIR               | WV-2     | 0.3m | 1      | 650²     |                    |
| PAN       | PAN               | WV-2     | 0.3m | 1      | 650²     |                    |
| MS        | MS                | WV-2     | 1.2m | 8      | 163²     |                    |
| **✅选用** | building polygons | 人工标注 | —    | —      | —        | **ControlNet条件** |

**SpaceNet 2关键数据**：685,000+建筑足迹，4 AOIs(Vegas 151K/Paris 24K/Shanghai 92K/Khartoum 36K)，200m×200m tiles

**处理步骤**：

1. **模态筛选**：保留全部模态（RGB、NIR、PAN、MS）+ building polygons
2. **格式读取**：影像 GeoTIFF，用 `tifffile.imread()` 读取，确认 650²；building polygons GeoJSON，用 `geopandas.read_file()` 读取
3. **云筛选**：无需筛选，已精心筛选

------

### SpaceNet 3（PAN补充，road centerline）

| 数据      | 模态            | 传感器    | GSD  | 波段数 | 原生尺寸 | 备注                        |
| --------- | --------------- | --------- | ---- | ------ | -------- | --------------------------- |
| RGB       | RGB             | WV-2/WV-3 | 0.3m | 3      | 650²     |                             |
| NIR       | NIR             | WV-2/WV-3 | 0.3m | 1      | 650²     |                             |
| PAN       | PAN             | WV-2/WV-3 | 0.3m | 1      | 650²     |                             |
| MS        | MS              | WV-2/WV-3 | 1.2m | 8      | 163²     |                             |
| **✅选用** | road centerline | 人工标注  | —    | —      | —        | **ControlNet条件**; 7类道路 |
| metadata  | OSM元数据       | QA/QC     | —    | —      | —        | →VLM上下文                  |

**SpaceNet 3关键数据**：8,677km道路，4 AOIs(Vegas 3685km/Paris 425km/Shanghai 3537km/Khartoum 1030km)，7类道路(Motorway/Primary/Secondary/Tertiary/Residential/Unclassified/Cart track)

**处理步骤**：

1. **模态筛选**：保留全部模态（RGB、NIR、PAN、MS）+ road centerline（7类道路）
2. **格式读取**：影像 GeoTIFF，用 `tifffile.imread()` 读取，确认 650²；road centerline GeoJSON，用 `geopandas.read_file()` 读取
3. **云筛选**：无需筛选，已精心筛选

------

### SpaceNet 4（PAN补充，off-nadir building）

| 数据      | 模态              | 传感器   | GSD        | 波段数 | 原生尺寸 | 备注                             |
| --------- | ----------------- | -------- | ---------- | ------ | -------- | -------------------------------- |
| RGB       | RGB               | WV-2     | 0.48-1.67m | 3      | 900²     | 27张影像，7-54°离底点角          |
| NIR       | NIR               | WV-2     | 0.48-1.67m | 1      | 900²     |                                  |
| PAN       | PAN               | WV-2     | 0.48-1.67m | 1      | 900²     |                                  |
| MS        | MS                | WV-2     | 1.2m       | 8      | 225²     |                                  |
| **✅选用** | building polygons | 人工标注 | —          | —      | —        | **ControlNet条件**; 120,000+足迹 |

**SpaceNet 4关键数据**：Atlanta, GA，27张WV-2影像(2009-12-22)，450m×450m tiles，3段离底点角(Nadir 0-25°/Off-Nadir 26-40°/Very Off-Nadir 40-55°)

**处理步骤**：

1. **模态筛选**：保留全部模态（RGB、NIR、PAN、MS）+ building polygons
2. **格式读取**：影像 GeoTIFF，用 `tifffile.imread()` 读取，确认 900²；building polygons GeoJSON，用 `geopandas.read_file()` 读取
3. **云筛选**：无需筛选，已精心筛选
4. **离底点角**：27张影像覆盖 7-54°，分3段（Nadir 0-25°/Off-Nadir 26-40°/Very Off-Nadir 40-55°），可作为额外条件信息

------

### SpaceNet 5（PAN补充，road network）

| 数据              | 模态            | 传感器   | GSD  | 波段数 | 原生尺寸 | 备注               |
| ----------------- | --------------- | -------- | ---- | ------ | -------- | ------------------ |
| RGB               | RGB             | WV-3     | 0.3m | 3      | 1300²    | ~400m×400m tiles   |
| NIR               | NIR             | WV-3     | 0.3m | 1      | 1300²    |                    |
| PAN               | PAN             | WV-3     | 0.3m | 1      | 1300²    |                    |
| MS                | MS              | WV-3     | 1.2m | 8      | 325²     |                    |
| **✅选用**         | road centerline | 人工标注 | —    | —      | —        | **ControlNet条件** |
| building polygons | 建筑轮廓        | 人工标注 | —    | —      | —        | →VLM上下文         |
| travel time       | 通行时间        | OSM      | —    | —      | —        | →VLM上下文         |

**SpaceNet 5关键数据**：3 AOIs(Moscow/Mumbai/San Juan)，~400m×400m tiles

**处理步骤**：

1. **模态筛选**：保留 RGB、NIR、PAN、MS + road centerline + building polygons；travel time（OSM）保留为VLM上下文
2. **格式读取**：影像 GeoTIFF，用 `tifffile.imread()` 读取，确认 1300²；road centerline/building polygons GeoJSON，用 `geopandas.read_file()` 读取
3. **云筛选**：无需筛选，已精心筛选