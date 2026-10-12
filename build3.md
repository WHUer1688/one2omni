# 3模态对齐数据集完整列表（10种）

> 排序：尺寸(≥512²优先) → 条件类别数(segmap/bbox同等地位,类别多优先) → 标注来源(Z>G) → 分辨率 → 数量
> 裁剪：n = floor((S-512)/256)+1, 50%重叠

### 1. RGB↔NIR↔MS（29个数据集，总计 198.7M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                   | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ------------------------ | --------------- | ---------------- | ------ |
| ⭐1   | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 512²@0.2m                | 241K            | n=1              | 241K   |
| ⭐2   | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m               | 100K            | n=41, 41×41=1681 | 168.1M |
| ⭐3   | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                | 4K              | n=5, 5×5=25      | 112K   |
| 4    | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                 | 40K             | n=1              | 40K    |
| 5    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                 | 4K              | n=1              | 4K     |
| 6    | SpaceNet 5             | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                | 25K             | n=1, 1×1=1       | 25K    |
| 7    | SpaceNet 2             | building polygons: Z(人工标注)                               | 650²@30cm                | 24K             | n=1              | 24K    |
| 8    | SpaceNet 4             | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                | 62K             | n=2, 2×2=4       | 248K   |
| 9    | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G+C(多源人工+自动标注+OSM+众包) | 512²@10m                 | 856K            | pad              | 856K   |
| 10   | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                 | 180K            | pad              | 180K   |
| 11   | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                 | 285K            | pad              | 285K   |
| 12   | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): P(QC)(AI预标+人工检查);DEM: G(地理叠加);land cover: G(地理叠加) | 509²@10m                 | 49K             | pad              | 49K    |
| 13   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                 | 39K             | pad              | 39K    |
| 14   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): P(QC)(TerraClass专家分类+D | 128²@10m                 | 30K             | pad              | 30K    |
| 15   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): P(QC)(UNet预标+人 | RGB@25cm, S1/S2@10m      | 20K             | pad              | 20K    |
| 16   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): P(QC)(自动分类+专家审核+人工精修)      | 256²@10m                 | 20K             | pad              | 20K    |
| 17   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                 | 500             | pad              | 500    |
| 18   | MADS                   | segmap(LC): C(OSM众包);endmember labels(6类: pavement/low vegetation/soil/tree/roof | 1371×888@10m             | 1场景(1371x888) | pad              | 1      |
| 19   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                 | 13K             | pad              | 13K    |
| 20   | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加);biomass(ESA CCI): G(地理叠加);built-up(GHSL): G( | 448²@10m                 | 17.0M           | pad              | 17.0M  |
| 21   | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加);DEM(Copernicus 30m→10m插值): G(地理叠加);NDVI: R(规则生成) | 264²@10m                 | 9.1M            | pad              | 9.1M   |
| 22   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m | 549K            | pad              | 549K   |
| 23   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                 | 8K              | pad              | 8K     |
| 24   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                 | 6K              | pad              | 6K     |
| 25   | MajorTOM-Core          | cloud mask: P(模型伪标注)                                    | 1068²@10m                | 2               | n=3, 3×3=9       | 18     |
| 26   | SpaceNet 3             | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified | 650²@30cm                | 3K              | n=4, 4×4=16      | 59K    |
| 27   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                  | 400K            | pad              | 400K   |
| 28   | MMEarth                | DEM: G(地理叠加);冠层高度(ETH-GCHM): G(地理叠加);WorldCover: G(ESA,地理叠加);Biome/Ecoregion: G( | 128²@10m                 | 1.2M            | pad              | 1.2M   |
| 29   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                  | 1K              | pad              | 1K     |

### 2. RGB↔NIR↔PAN（6个数据集，总计 681K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                             | 原数量 | 裁剪        | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | ---------------------------------- | ------ | ----------- | ------ |
| ⭐1   | SpaceNet 5      | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                          | 25K    | n=1, 1×1=1  | 25K    |
| ⭐2   | SpaceNet 2      | building polygons: Z(人工标注)                               | 650²@30cm                          | 24K    | n=1         | 24K    |
| ⭐3   | SpaceNet 4      | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                          | 62K    | n=2, 2×2=4  | 248K   |
| 4    | SpaceNet 3      | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified | 650²@30cm                          | 3K     | n=4, 4×4=16 | 59K    |
| 5    | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | 900²@0.5m                          | 3K     | n=2, 2×2=4  | 13K    |
| 6    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | RGB/NIR: 512²@0.2m; PAN: 256²@0.4m | 241K   | n=1         | 241K   |

### 3. RGB↔NIR↔SAR（30个数据集，总计 200.4M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                   | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ------------------------ | --------------- | ---------------- | ------ |
| ⭐1   | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 512²@0.2m                | 241K            | n=1              | 241K   |
| ⭐2   | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m               | 100K            | n=41, 41×41=1681 | 168.1M |
| ⭐3   | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                | 4K              | n=5, 5×5=25      | 112K   |
| 4    | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                 | 40K             | n=1              | 40K    |
| 5    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                 | 4K              | n=1              | 4K     |
| 6    | GF-FloodNet            | segmap(水体/洪水, 像素级): Z(人工标注)                       | 256²@1.5-4m              | 13K             | pad              | 13K    |
| 7    | WHU-OPT-SAR            | segmap(7类: 农田/水体/村庄/城市/森林/道路/其他): Z(人工标注) | 256²@5m                  | 100             | pad              | 100    |
| 8    | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G+C(多源人工+自动标注+OSM+众包) | 512²@10m                 | 856K            | pad              | 856K   |
| 9    | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                 | 180K            | pad              | 180K   |
| 10   | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                 | 285K            | pad              | 285K   |
| 11   | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): P(QC)(AI预标+人工检查);DEM: G(地理叠加);land cover: G(地理叠加) | 509²@10m                 | 49K             | pad              | 49K    |
| 12   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                 | 39K             | pad              | 39K    |
| 13   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): P(QC)(TerraClass专家分类+D | 128²@10m                 | 30K             | pad              | 30K    |
| 14   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): P(QC)(UNet预标+人 | RGB@25cm, S1/S2@10m      | 20K             | pad              | 20K    |
| 15   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): P(QC)(自动分类+专家审核+人工精修)      | 256²@10m                 | 20K             | pad              | 20K    |
| 16   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                 | 500             | pad              | 500    |
| 17   | MADS                   | segmap(LC): C(OSM众包);endmember labels(6类: pavement/low vegetation/soil/tree/roof | 1371×888@10m             | 1场景(1371x888) | pad              | 1      |
| 18   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                 | 13K             | pad              | 13K    |
| 19   | C2Seg-AB               | segmap(13类地物): G(OSM叠加+人工检查)                        | 886²@10m                 | 413             | n=2, 2×2=4       | 1K     |
| 20   | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加);biomass(ESA CCI): G(地理叠加);built-up(GHSL): G( | 448²@10m                 | 17.0M           | pad              | 17.0M  |
| 21   | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加);DEM(Copernicus 30m→10m插值): G(地理叠加);NDVI: R(规则生成) | 264²@10m                 | 9.1M            | pad              | 9.1M   |
| 22   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m | 549K            | pad              | 549K   |
| 23   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                 | 8K              | pad              | 8K     |
| 24   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                 | 6K              | pad              | 6K     |
| 25   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                  | 400K            | pad              | 400K   |
| 27   | MMM-RS                 | text prompt(4组件信息丰富型,用于文本→图像生成):①图像内容描述: P+H(BLIP-2自动生成+人工筛查修正)②GSD等级(5级:超高/高/普通 | 512²@0.5-10m             | 2.1M            | n=1              | 2.1M   |
| 28   | SODAS                  | crop-type map(18类+休耕): G(LPIS/SIGPAC欧盟CAP农户申报记录); precipitation(日降雨): G(气象站) | 3587×3607@10m            | 301(S1)+210(S2) | n=13, 13×13=169  | 50K    |
| 29   | MMEarth                | DEM: G(地理叠加);冠层高度(ETH-GCHM): G(地理叠加);WorldCover: G(ESA,地理叠加);Biome/Ecoregion: G( | 128²@10m                 | 1.2M            | pad              | 1.2M   |
| 30   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                  | 1K              | pad              | 1K     |
| 31   | SpaceNet 6 MSAW        | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | 900²@0.5m                | 3K              | n=2, 2×2=4       | 13K    |

### 4. RGB↔MS↔PAN（6个数据集，总计 681K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                                             | 原数量 | 裁剪        | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | -------------------------------------------------- | ------ | ----------- | ------ |
| ⭐1   | SpaceNet 5      | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                                          | 25K    | n=1, 1×1=1  | 25K    |
| ⭐2   | SpaceNet 2      | building polygons: Z(人工标注)                               | 650²@30cm                                          | 24K    | n=1         | 24K    |
| ⭐3   | SpaceNet 4      | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                                          | 62K    | n=2, 2×2=4  | 248K   |
| 4    | SpaceNet 3      | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified | 650²@30cm                                          | 3K     | n=4, 4×4=16 | 59K    |
| 5    | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | RGB/PAN: 900²@0.5m; MS: 225²@2.0m                  | 3K     | n=2, 2×2=4  | 13K    |
| 6    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | RGB: 512²@0.2m; PAN: 256²@0.4m; MS(SPOT): 64²@1.6m | 241K   | n=1         | 241K   |

### 5. RGB↔MS↔SAR（25个数据集，总计 198.3M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                        | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ----------------------------- | --------------- | ---------------- | ------ |
| ⭐1   | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m                    | 100K            | n=41, 41×41=1681 | 168.1M |
| ⭐2   | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                     | 4K              | n=5, 5×5=25      | 112K   |
| ⭐3   | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                      | 40K             | n=1              | 40K    |
| 4    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                      | 4K              | n=1              | 4K     |
| 5    | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 10²@10m(Sentinel)             | 241K            | pad              | 241K   |
| 6    | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G+C(多源人工+自动标注+OSM+众包) | 512²@10m                      | 856K            | pad              | 856K   |
| 7    | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                      | 180K            | pad              | 180K   |
| 8    | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                      | 285K            | pad              | 285K   |
| 9    | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): P(QC)(AI预标+人工检查);DEM: G(地理叠加);land cover: G(地理叠加) | 509²@10m                      | 49K             | pad              | 49K    |
| 10   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                      | 39K             | pad              | 39K    |
| 11   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): P(QC)(TerraClass专家分类+D | 128²@10m                      | 30K             | pad              | 30K    |
| 12   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): P(QC)(UNet预标+人 | RGB@25cm, S1/S2@10m           | 20K             | pad              | 20K    |
| 13   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): P(QC)(自动分类+专家审核+人工精修)      | 256²@10m                      | 20K             | pad              | 20K    |
| 14   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                      | 500             | pad              | 500    |
| 15   | MADS                   | segmap(LC): C(OSM众包);endmember labels(6类: pavement/low vegetation/soil/tree/roof | 1371×888@10m                  | 1场景(1371x888) | pad              | 1      |
| 16   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                      | 13K             | pad              | 13K    |
| 17   | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加);biomass(ESA CCI): G(地理叠加);built-up(GHSL): G( | 448²@10m                      | 17.0M           | pad              | 17.0M  |
| 18   | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加);DEM(Copernicus 30m→10m插值): G(地理叠加);NDVI: R(规则生成) | 264²@10m                      | 9.1M            | pad              | 9.1M   |
| 19   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m      | 549K            | pad              | 549K   |
| 20   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                      | 8K              | pad              | 8K     |
| 21   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                      | 6K              | pad              | 6K     |
| 22   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                       | 400K            | pad              | 400K   |
| 23   | MMEarth                | DEM: G(地理叠加);冠层高度(ETH-GCHM): G(地理叠加);WorldCover: G(ESA,地理叠加);Biome/Ecoregion: G( | 128²@10m                      | 1.2M            | pad              | 1.2M   |
| 24   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                       | 1K              | pad              | 1K     |
| 25   | SpaceNet 6 MSAW        | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | RGB: 900²@0.5m; MS: 225²@2.0m | 3K              | n=2, 2×2=4       | 13K    |

### 6. RGB↔PAN↔SAR（2个数据集，总计 254K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                                              | 原数量 | 裁剪       | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | --------------------------------------------------- | ------ | ---------- | ------ |
| ⭐1   | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | 900²@0.5m                                           | 3K     | n=2, 2×2=4 | 13K    |
| 2    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | RGB: 512²@0.2m; PAN: 256²@0.4m; SAR(S1): 10²@10.24m | 241K   | n=1        | 241K   |

### 7. NIR↔MS↔PAN（6个数据集，总计 681K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                                             | 原数量 | 裁剪        | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | -------------------------------------------------- | ------ | ----------- | ------ |
| ⭐1   | SpaceNet 5      | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                                          | 25K    | n=1, 1×1=1  | 25K    |
| ⭐2   | SpaceNet 2      | building polygons: Z(人工标注)                               | 650²@30cm                                          | 24K    | n=1         | 24K    |
| ⭐3   | SpaceNet 4      | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                                          | 62K    | n=2, 2×2=4  | 248K   |
| 4    | SpaceNet 3      | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified | 650²@30cm                                          | 3K     | n=4, 4×4=16 | 59K    |
| 5    | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | NIR/PAN: 900²@0.5m; MS: 225²@2.0m                  | 3K     | n=2, 2×2=4  | 13K    |
| 6    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | NIR: 512²@0.2m; PAN: 256²@0.4m; MS(SPOT): 64²@1.6m | 241K   | n=1         | 241K   |

### 8. NIR↔MS↔SAR（25个数据集，总计 198.3M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                        | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ----------------------------- | --------------- | ---------------- | ------ |
| ⭐1   | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m                    | 100K            | n=41, 41×41=1681 | 168.1M |
| ⭐2   | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                     | 4K              | n=5, 5×5=25      | 112K   |
| ⭐3   | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                      | 40K             | n=1              | 40K    |
| 4    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                      | 4K              | n=1              | 4K     |
| 5    | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 10²@10m(Sentinel)             | 241K            | pad              | 241K   |
| 6    | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G+C(多源人工+自动标注+OSM+众包) | 512²@10m                      | 856K            | pad              | 856K   |
| 7    | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                      | 180K            | pad              | 180K   |
| 8    | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                      | 285K            | pad              | 285K   |
| 9    | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): P(QC)(AI预标+人工检查);DEM: G(地理叠加);land cover: G(地理叠加) | 509²@10m                      | 49K             | pad              | 49K    |
| 10   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                      | 39K             | pad              | 39K    |
| 11   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): P(QC)(TerraClass专家分类+D | 128²@10m                      | 30K             | pad              | 30K    |
| 12   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): P(QC)(UNet预标+人 | RGB@25cm, S1/S2@10m           | 20K             | pad              | 20K    |
| 13   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): P(QC)(自动分类+专家审核+人工精修)      | 256²@10m                      | 20K             | pad              | 20K    |
| 14   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                      | 500             | pad              | 500    |
| 15   | MADS                   | segmap(LC): C(OSM众包);endmember labels(6类: pavement/low vegetation/soil/tree/roof | 1371×888@10m                  | 1场景(1371x888) | pad              | 1      |
| 16   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                      | 13K             | pad              | 13K    |
| 17   | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加);biomass(ESA CCI): G(地理叠加);built-up(GHSL): G( | 448²@10m                      | 17.0M           | pad              | 17.0M  |
| 18   | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加);DEM(Copernicus 30m→10m插值): G(地理叠加);NDVI: R(规则生成) | 264²@10m                      | 9.1M            | pad              | 9.1M   |
| 19   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m      | 549K            | pad              | 549K   |
| 20   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                      | 8K              | pad              | 8K     |
| 21   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                      | 6K              | pad              | 6K     |
| 22   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                       | 400K            | pad              | 400K   |
| 23   | MMEarth                | DEM: G(地理叠加);冠层高度(ETH-GCHM): G(地理叠加);WorldCover: G(ESA,地理叠加);Biome/Ecoregion: G( | 128²@10m                      | 1.2M            | pad              | 1.2M   |
| 24   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                       | 1K              | pad              | 1K     |
| 25   | SpaceNet 6 MSAW        | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | NIR: 900²@0.5m; MS: 225²@2.0m | 3K              | n=2, 2×2=4       | 13K    |

### 9. NIR↔PAN↔SAR（2个数据集，总计 254K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                                              | 原数量 | 裁剪       | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | --------------------------------------------------- | ------ | ---------- | ------ |
| ⭐1   | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | 900²@0.5m                                           | 3K     | n=2, 2×2=4 | 13K    |
| 2    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | NIR: 512²@0.2m; PAN: 256²@0.4m; SAR(S1): 10²@10.24m | 241K   | n=1        | 241K   |

### 10. MS↔PAN↔SAR（2个数据集，总计 254K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                                                  | 原数量 | 裁剪       | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | ------------------------------------------------------- | ------ | ---------- | ------ |
| ⭐1   | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | PAN: 900²@0.5m; MS: 225²@2.0m                           | 3K     | n=2, 2×2=4 | 13K    |
| 2    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | PAN: 256²@0.4m; MS(SPOT): 64²@1.6m; SAR(S1): 10²@10.24m | 241K   | pad        | 241K   |