# 2模态对齐数据集完整列表

> 排序规则：条件类型(segbmap>bbox>text/scene) → 标注来源(Z>G) → 尺寸(≥512²优先) → 分辨率 → 数量
> 裁剪公式：n = floor((S-512)/256)+1，patches/张 = n×n

------

## 2模态对齐（10种）

### 1. RGB↔NIR（47个数据集，总计 203.6M）

| 排名 | 数据集                             | 条件                                                         | 原尺寸                   | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------------------- | ------------------------------------------------------------ | ------------------------ | --------------- | ---------------- | ------ |
| 1    | FLAIR-HUB                          | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 512²@0.2m                | 241K            | n=1              | 241K   |
| 2    | GF-7 Building                      | building polygons(instance+semantic): Z(人工标注)            | 512²@0.65m               | 5K              | n=1              | 5K     |
| 3    | CatLC                              | segmap(41类LC): Z(专家photointerpretation)                   | 960²@1m                  | 34K             | n=2, 2×2=4       | 136K   |
| 4    | DynamicEarthNet                    | segmap(7类SCS: 不透水面/农业/森林植被/湿地/裸土/水体/雪冰): Z(月度人工标注, 24月标注/每AOI) | 1024²@3m                 | 54K             | n=3, 3×3=9       | 492K   |
| 5    | GID / GID-15                       | segmap(GID-5 5类 / GID-15 24子类): Z(人工标注)               | 6800×7200@4m             | 5               | n=27, 27×27=729  | 3K     |
| 6    | S1S2-Water                         | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m               | 100K            | n=41, 41×41=1681 | 168.1M |
| 7    | AirMOS                             | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                | 4K              | n=5, 5×5=25      | 112K   |
| 8    | Agriculture-Vision                 | segmap(9类农业异常模式): Z(人工标注); 支持农业异常检测       | 512²@10cm                | 94K             | n=1              | 94K    |
| 9    | CloudLULC-Set                      | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                 | 40K             | n=1              | 40K    |
| 10   | Sen1Floods11                       | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                 | 4K              | n=1              | 4K     |
| 11   | SpaceNet 5                         | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                | 25K             | n=1, 1×1=1       | 25K    |
| 12   | SpaceNet 2                         | building polygons: Z(人工标注)                               | 650²@30cm                | 24K             | n=1              | 24K    |
| 13   | SpaceNet 4                         | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                | 62K             | n=2, 2×2=4       | 248K   |
| 14   | C2Seg-AB                           | segmap(13类地物): G(OSM叠加+人工检查)                        | 886²@10m                 | 413             | n=2, 2×2=4       | 1K     |
| 15   | MajorTOM-Core                      | cloud mask: P(模型伪标注)                                    | 1068²@10m                | 2               | n=3, 3×3=9       | 18     |
| 16   | MMShip                             | HBB bbox(1类舰船): Z(人工)                                   | 512²@10m                 | 5K              | n=1              | 5K     |
| 17   | VEDAI                              | OBB bbox(9类车辆/目标): Z(人工, 含旋转角度4角点标注)         | 1024²@12.5cm             | 1K              | n=3, 3×3=9       | 11K    |
| 18   | SpaceNet 3                         | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified/Cart Track + surface/bridge/lane属性): Z(标注精度≤7px, QA/QC双检); ; metadata(OSM标签体系): Z(人工标注) | 650²@30cm                | 3K              | n=4, 4×4=16      | 59K    |
| 19   | SODAS                              | crop-type map(18类+休耕): G(LPIS/SIGPAC欧盟CAP农户申报记录); precipitation(日降雨): G(气象站) | 3587×3607@10m            | 301(S1)+210(S2) | n=13, 13×13=169  | 50K    |
| 20   | MMM-RS                             | text prompt(4组件信息丰富型,用于文本→图像生成):; ①图像内容描述: P+H(BLIP-2自动生成+人工筛查修正); ②GSD等级(5级:超高/高/普通/低/超低分辨率): R(原GSD按resize比例计算); ③天气(正常/雾/雪/低光): P(部分样本合成); ④卫星类型: Z(源数据集继承) | 512²@0.5-10m             | 2.1M            | n=1              | 2.1M   |
| —    | **<512² 数据集**                   |                                                              |                          |                 |                  |        |
| 21   | CloudSEN12                         | cloud mask(厚云/薄云/云影, 人工标注): Z(QC)(AI预标+人工检查); ; DEM: G(地理叠加); ; land cover: G(地理叠加) | 509²@10m                 | 49K             | pad              | 49K    |
| 22   | M3LEO                              | segmap(ESA WorldCover 11类): G(地理叠加); ; biomass(ESA CCI): G(地理叠加); ; built-up(GHSL): G(地理叠加); ; veg continuous(MODIS): G(地理叠加); ; DEM(SRTM 90m): G(地理叠加) | 448²@10m                 | 17.0M           | pad              | 17.0M  |
| 23   | 38-Cloud                           | cloud mask: R+Z(规则生成+人工标注:FMask阈值+人工修正)(人工云/非云标签) | 384²@30m                 | 17K             | pad              | 17K    |
| 24   | TerraMesh                          | segmap(ESRI全球土地覆盖10m): G(地理叠加); ; DEM(Copernicus 30m→10m插值): G(地理叠加); ; NDVI: R(规则生成) | 264²@10m                 | 9.1M            | pad              | 9.1M   |
| 25   | GF-FloodNet                        | segmap(水体/洪水, 像素级): Z(人工标注)                       | 256²@1.5-4m              | 13K             | pad              | 13K    |
| 26   | EarthScape                         | segmap(7类地表地质): Z(人工标注);DEM: G(地理叠加)            | 256²@1.52m               | 31K             | pad              | 31K    |
| 27   | WHU-OPT-SAR                        | segmap(7类: 农田/水体/村庄/城市/森林/道路/其他): Z(人工标注) | 256²@5m                  | 100             | pad              | 100    |
| 28   | SEN12MS                            | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                 | 180K            | pad              | 180K   |
| 29   | OlmoEarth                          | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                 | 285K            | pad              | 285K   |
| 30   | LuojiaSET-OSFCR                    | segmap(土地覆盖分类): Z(QC)(自动分类+专家审核+人工精修)      | 256²@10m                 | 20K             | pad              | 20K    |
| 31   | HunanMultimodalDataset             | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                 | 500             | pad              | 500    |
| 32   | Plot-Rice v1.0                     | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                 | 13K             | pad              | 13K    |
| 33   | MultiSenGE                         | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                 | 8K              | pad              | 8K     |
| 34   | DFC20 / DFC2020                    | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                 | 6K              | pad              | 6K     |
| 35   | fMoW (Functional Map of the World) | HBB bbox(62类功能性用地分类): Z(人工标注)                    | 256²@0.3-5m              | 1.0M            | pad              | 1.0M   |
| 36   | TalloS                             | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                  | 1K              | pad              | 1K     |
| 37   | Sen12Landslides                    | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                 | 39K             | pad              | 39K    |
| 38   | CerraData-4MM                      | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): Z(QC)(TerraClass专家分类+D | 128²@10m                 | 30K             | pad              | 30K    |
| 39   | BigEarthNet                        | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m | 549K            | pad              | 549K   |
| 40   | MMEarth                            | DEM: G(地理叠加); ; 冠层高度(ETH-GCHM): G(地理叠加); ; WorldCover: G(ESA,地理叠加); ; Biome/Ecoregion: G(地理叠加); ; ERA5温/降水: G(地理叠加); Dynamic World 9类LandCover: G | 128²@10m                 | 1.2M            | pad              | 1.2M   |
| 41   | So2Sat LCZ42                       | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                  | 400K            | pad              | 400K   |
| 42   | SAT-4                              | scene label(4类): Z(人工标注)                                | 28²@1.03m                | 500K            | pad              | 500K   |
| 43   | SAT-6                              | scene label(6类): Z(人工标注)                                | 28²@1.03m                | 405K            | pad              | 405K   |
| 44   | DeepFlood                          | segmap(4类: open water/dry vegetation/inundated vegetation/other): Z(QC)(UNet预标+人工修正, ArcGIS Pro); ; DEM+slope: G(地理叠加) | RGB@25cm, S1/S2@10m      | 20K             | pad              | 20K    |
| 45   | SatlasPretrain                     | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G(多源人工+自动标注+OSM+众包) | 512²@10m                 | 856K            | pad              | 856K   |
| 46   | MADS                               | segmap(LC): C(OSM众包); ; endmember labels(6类: pavement/low vegetation/soil/tree/roof/water): Z(人工标注); ; DSM: G(地理叠加); ; GIS: G(地理叠加) | 1371×888@10m             | 1场景(1371x888) | pad              | 1      |
| 47   | SpaceNet 6 MSAW                    | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | 900²@0.5m                | 3K              | n=2, 2×2=4       | 13K    |

### 2. RGB↔MS（31个数据集，总计 198.8M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                        | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ----------------------------- | --------------- | ---------------- | ------ |
| 1    | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m                    | 100K            | n=41, 41×41=1681 | 168.1M |
| 2    | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                     | 4K              | n=5, 5×5=25      | 112K   |
| 3    | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                      | 40K             | n=1              | 40K    |
| 4    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                      | 4K              | n=1              | 4K     |
| 5    | SpaceNet 5             | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                     | 25K             | n=1, 1×1=1       | 25K    |
| 6    | SpaceNet 2             | building polygons: Z(人工标注)                               | 650²@30cm                     | 24K             | n=1              | 24K    |
| 7    | SpaceNet 4             | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                     | 62K             | n=2, 2×2=4       | 248K   |
| 8    | MajorTOM-Core          | cloud mask: P(模型伪标注)                                    | 1068²@10m                     | 2               | n=3, 3×3=9       | 18     |
| 9    | xView                  | HBB bbox(60类7父类): C(单一设施众包标注, 3阶段QC: 人工→监督→金标准, precision≥0.75 & recall≥0.95 @0.5 IoU) | 3320²@0.3m                    | 847             | n=10, 10×10=100  | 85K    |
| 10   | SpaceNet 3             | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified/Cart Track + surface/bridge/lane属性): Z(标注精度≤7px, QA/QC双检); ; metadata(OSM标签体系): Z(人工标注) | 650²@30cm                     | 3K              | n=4, 4×4=16      | 59K    |
| —    | **<512² 数据集**       |                                                              |                               |                 |                  |        |
| 11   | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): Z(QC)(AI预标+人工检查); ; DEM: G(地理叠加); ; land cover: G(地理叠加) | 509²@10m                      | 49K             | pad              | 49K    |
| 12   | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加); ; biomass(ESA CCI): G(地理叠加); ; built-up(GHSL): G(地理叠加); ; veg continuous(MODIS): G(地理叠加); ; DEM(SRTM 90m): G(地理叠加) | 448²@10m                      | 17.0M           | pad              | 17.0M  |
| 13   | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加); ; DEM(Copernicus 30m→10m插值): G(地理叠加); ; NDVI: R(规则生成) | 264²@10m                      | 9.1M            | pad              | 9.1M   |
| 14   | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                      | 180K            | pad              | 180K   |
| 15   | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                      | 285K            | pad              | 285K   |
| 16   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): Z(QC)(自动分类+专家审核+人工精修)      | 256²@10m                      | 20K             | pad              | 20K    |
| 17   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                      | 500             | pad              | 500    |
| 18   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                      | 13K             | pad              | 13K    |
| 19   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                      | 8K              | pad              | 8K     |
| 20   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                      | 6K              | pad              | 6K     |
| 21   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                       | 1K              | pad              | 1K     |
| 22   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                      | 39K             | pad              | 39K    |
| 23   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): Z(QC)(TerraClass专家分类+D | 128²@10m                      | 30K             | pad              | 30K    |
| 24   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m      | 549K            | pad              | 549K   |
| 25   | MMEarth                | DEM: G(地理叠加); ; 冠层高度(ETH-GCHM): G(地理叠加); ; WorldCover: G(ESA,地理叠加); ; Biome/Ecoregion: G(地理叠加); ; ERA5温/降水: G(地理叠加); Dynamic World 9类LandCover: G | 128²@10m                      | 1.2M            | pad              | 1.2M   |
| 26   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                       | 400K            | pad              | 400K   |
| 27   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): Z(QC)(UNet预标+人工修正, ArcGIS Pro); ; DEM+slope: G(地理叠加) | RGB@25cm, S1/S2@10m           | 20K             | pad              | 20K    |
| 28   | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 10²@10m(Sentinel)             | 241K            | n=1              | 241K   |
| 29   | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G(多源人工+自动标注+OSM+众包) | 512²@10m                      | 856K            | pad              | 856K   |
| 30   | MADS                   | segmap(LC): C(OSM众包); ; endmember labels(6类: pavement/low vegetation/soil/tree/roof/water): Z(人工标注); ; DSM: G(地理叠加); ; GIS: G(地理叠加) | 1371×888@10m                  | 1场景(1371x888) | pad              | 1      |
| 31   | SpaceNet 6 MSAW        | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | RGB: 900²@0.5m; MS: 225²@2.0m | 3K              | n=2, 2×2=4       | 13K    |

### 3. RGB↔PAN（7个数据集，总计 688K）

| 排名 | 数据集           | 条件                                                         | 原尺寸                         | 原数量 | 裁剪        | 512²后 |
| ---- | ---------------- | ------------------------------------------------------------ | ------------------------------ | ------ | ----------- | ------ |
| 1    | SpaceNet 5       | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                      | 25K    | n=1, 1×1=1  | 25K    |
| 2    | SpaceNet 2       | building polygons: Z(人工标注)                               | 650²@30cm                      | 24K    | n=1         | 24K    |
| 3    | SpaceNet 4       | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                      | 62K    | n=2, 2×2=4  | 248K   |
| 4    | SpaceNet 3       | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified/Cart Track + surface/bridge/lane属性): Z(标注精度≤7px, QA/QC双检); ; metadata(OSM标签体系): Z(人工标注) | 650²@30cm                      | 3K     | n=4, 4×4=16 | 59K    |
| —    | **<512² 数据集** |                                                              |                                |        |             |        |
| 5    | SpaceNet 1       | building polygons: Z(人工标注)                               | 400²@50cm                      | 6K     | pad         | 6K     |
| 6    | SpaceNet 6 MSAW  | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | 900²@0.5m                      | 3K     | n=2, 2×2=4  | 13K    |
| 7    | FLAIR-HUB        | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | RGB: 512²@0.2m; PAN: 256²@0.4m | 241K   | n=1         | 241K   |

### 4. RGB↔SAR（53个数据集，总计 214.0M）

| 排名 | 数据集                          | 条件                                                         | 原尺寸                         | 原数量          | 裁剪             | 512²后 |
| ---- | ------------------------------- | ------------------------------------------------------------ | ------------------------------ | --------------- | ---------------- | ------ |
| 1    | BRIGHT                          | building damage mask(3类: Intact/Damaged/Destroyed + Background = 4类): Z(EO专家像素级损 | 1024²@0.3-1m                   | 4K              | n=3, 3×3=9       | 38K    |
| 2    | UBCv2                           | polygon roof type(12类): Z(人工标注); building polygons: Z(人工标注) | 512²@0.5m                      | 5K              | n=1              | 5K     |
| 3    | EarthMiss                       | segmap(8类: 背景/建筑/道路/水体/裸地/森林/农田/操场): Z(人工标注+跨模态对齐) | 1024²@0.6m                     | 3K              | n=3, 3×3=9       | 30K    |
| 4    | Globe230k                       | segmap(10类LULC): Z(人工标注)                                | 512²@1m                        | 232K            | n=1              | 232K   |
| 5    | Multi-Complex-Seg               | segmap(7类: 道路/农田/草地/森林/水体/建筑/其他): Z(人工交叉验证标注) | 512²@1m                        | 8K              | n=1              | 8K     |
| 6    | S1S2-Water                      | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m                     | 100K            | n=41, 41×41=1681 | 168.1M |
| 7    | AirMOS                          | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                      | 4K              | n=5, 5×5=25      | 112K   |
| 8    | CloudLULC-Set                   | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                       | 40K             | n=1              | 40K    |
| 9    | Sen1Floods11                    | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                       | 4K              | n=1              | 4K     |
| 10   | DFC2025                         | segmap(8类LC): P(OpenEarthMap模型伪标注); eval:Z(专家标注); ; building damage(4类): Z(Track2人工标注) | 1024²@0.15-0.5m                | 7K              | n=3, 3×3=9       | 65K    |
| 11   | OpenEarthMap-SAR                | segmap(8类: 建筑/道路/水体/树木/农业用地/牧场/裸地/已开发空间): Z+P(部分人工标注+全量伪标签扩充) | 1024²@0.15-0.5m                | 5K              | n=3, 3×3=9       | 45K    |
| 12   | DisasterM3                      | segmap(指代分割+建筑掩码+道路估计, 像素级): Z(QC)(源mask来自人工标注xBD+BRIGHT, GPT-4o预生成+灾害专家验证); ; HBB bbox(12类承灾体+空间关系): Z+P(专家标注+GPT-4o生成关系)(专家标注BBR/ORR); ; text prompt(123,010条/9任务/5层级, 全部预生成自带): P+H(GPT-4o生成变体, 多位专家标注答案+起草报告, 计数类由mask保证精确) | 1024²@0.8m                     | 26K             | n=3, 3×3=9       | 242K   |
| 13   | GeoNRW                          | segmap(10类LULC): G(州开放数据+OSM);DEM: G(地理叠加)         | 1000²@1m                       | 7K              | n=2, 2×2=4       | 31K    |
| 14   | C2Seg-AB                        | segmap(13类地物): G(OSM叠加+人工检查)                        | 886²@10m                       | 413             | n=2, 2×2=4       | 1K     |
| 15   | MOS-Ship                        | OBB(船舶): Z(人工标注); 合成退化基准含云/雾模拟(评估恶劣天气鲁棒性) | 1024²@0.75m                    | 1K              | n=3, 3×3=9       | 11K    |
| 17   | MMM-RS                          | text prompt(4组件信息丰富型,用于文本→图像生成):; ①图像内容描述: P+H(BLIP-2自动生成+人工筛查修正); ②GSD等级(5级:超高/高/普通/低/超低分辨率): R(原GSD按resize比例计算); ③天气(正常/雾/雪/低光): P(部分样本合成); ④卫星类型: Z(源数据集继承) | 512²@varied                    | 2.1M            | n=1              | 2.1M   |
| 18   | MEOW                            | DEM: G(TanDEM-X连续高程值); ; land cover: G(ESA WorldCover); ; 地理网格: G(地理叠加) | 4096²@10m                      | 46K             | n=15, 15×15=225  | 10.5M  |
| 19   | SODAS                           | crop-type map(18类+休耕): G(LPIS/SIGPAC欧盟CAP农户申报记录); precipitation(日降雨): G(气象站) | 3587×3607@10m                  | 301(S1)+210(S2) | n=13, 13×13=169  | 50K    |
| 20   | SOMA-1M                         | 12类地物标签(scene-level): P+H(自动分配+人工检查)            | 512²@varied                    | 1.3M            | n=1              | 1.3M   |
| 21   | M4-SAR                          | OBB(6类: 桥梁/港口/油罐/操场/机场/风机): P(QC)(模型伪标注+人工修正) | 512²@10m                       | 112K            | n=1              | 112K   |
| —    | **<512² 数据集**                |                                                              |                                |                 |                  |        |
| 21   | CloudSEN12                      | cloud mask(厚云/薄云/云影, 人工标注): Z(QC)(AI预标+人工检查); ; DEM: G(地理叠加); ; land cover: G(地理叠加) | 509²@10m                       | 49K             | pad              | 49K    |
| 22   | M3LEO                           | segmap(ESA WorldCover 11类): G(地理叠加); ; biomass(ESA CCI): G(地理叠加); ; built-up(GHSL): G(地理叠加); ; veg continuous(MODIS): G(地理叠加); ; DEM(SRTM 90m): G(地理叠加) | 448²@10m                       | 17.0M           | pad              | 17.0M  |
| 23   | M²Change                        | segmap(建筑损毁二值变化检测): Z(人工标注)(UNOSAT+OSM+专家验证) | 400²@1m                        | 5K              | pad              | 5K     |
| 24   | TerraMesh                       | segmap(ESRI全球土地覆盖10m): G(地理叠加); ; DEM(Copernicus 30m→10m插值): G(地理叠加); ; NDVI: R(规则生成) | 264²@10m                       | 9.1M            | pad              | 9.1M   |
| 25   | PIE-RGB-SAR                     | segmap(6类LULC): Z(人工标注)                                 | 256²@0.5m(RGB)/3m(SAR)         | 4K              | pad              | 4K     |
| 26   | YESeg-OPT-SAR                   | segmap(8类): Z(人工标注)                                     | 256²@0.5m                      | 2K              | pad              | 2K     |
| 27   | GF-FloodNet                     | segmap(水体/洪水, 像素级): Z(人工标注)                       | 256²@1.5-4m                    | 13K             | pad              | 13K    |
| 28   | WHU-OPT-SAR                     | segmap(7类: 农田/水体/村庄/城市/森林/道路/其他): Z(人工标注) | 256²@5m                        | 100             | pad              | 100    |
| 29   | SEN12MS                         | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                       | 180K            | pad              | 180K   |
| 30   | OlmoEarth                       | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                       | 285K            | pad              | 285K   |
| 31   | LuojiaSET-OSFCR                 | segmap(土地覆盖分类): Z(QC)(自动分类+专家审核+人工精修)      | 256²@10m                       | 20K             | pad              | 20K    |
| 32   | HunanMultimodalDataset          | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                       | 500             | pad              | 500    |
| 33   | Plot-Rice v1.0                  | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                       | 13K             | pad              | 13K    |
| 34   | MultiSenGE                      | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                       | 8K              | pad              | 8K     |
| 35   | DFC20 / DFC2020                 | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                       | 6K              | pad              | 6K     |
| 36   | Wuhan SAR-optical CD pair       | change mask(逐像素变化/未变化): D+R(mask派生+规则生成:算法+阈值自动生成) | 256²@3m                        | 793             | pad              | 793    |
| 37   | OGSOD-2.0(论文付费，无法看全文) | OBB(4类: bridge/harbor/oil tank/playground): Z(人工实例标注) | 256²@3m                        | 20K             | pad              | 20K    |
| 38   | SEN1-2                          | scene label(12类): Z(人工标注)                               | 256²@10m                       | 282K            | pad              | 282K   |
| 39   | 3MOS                            | 8类场景标签(城市/农村/平原/丘陵/山地/水域/沙漠/冻土): G(按场景类型分类) | 512²@1.25m, 256²@≥2.5m         | 113K            | pad              | 113K   |
| 40   | TalloS                          | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                        | 1K              | pad              | 1K     |
| 41   | Sen12Landslides                 | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                       | 39K             | pad              | 39K    |
| 42   | CerraData-4MM                   | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): Z(QC)(TerraClass专家分类+D | 128²@10m                       | 30K             | pad              | 30K    |
| 43   | BigEarthNet                     | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m       | 549K            | pad              | 549K   |
| 44   | MMEarth                         | DEM: G(地理叠加); ; 冠层高度(ETH-GCHM): G(地理叠加); ; WorldCover: G(ESA,地理叠加); ; Biome/Ecoregion: G(地理叠加); ; ERA5温/降水: G(地理叠加); Dynamic World 9类LandCover: G | 128²@10m                       | 1.2M            | pad              | 1.2M   |
| 45   | So2Sat LCZ42                    | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                        | 400K            | pad              | 400K   |
| 46   | DeepFlood                       | segmap(4类: open water/dry vegetation/inundated vegetation/other): Z(QC)(UNet预标+人工修正, ArcGIS Pro); ; DEM+slope: G(地理叠加) | RGB@25cm, S1/S2@10m            | 20K             | pad              | 20K    |
| 47   | FLAIR-HUB                       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 10²@10m(Sentinel)              | 241K            | n=1              | 241K   |
| 48   | OGSOD-1.0                       | HBB(3类: 桥梁/港口/油罐): Z(人工标注)                        | 256²@3m(SAR)/10m(光学)         | 14K             | pad              | 14K    |
| 49   | MVSDA                           | HBB bbox: Z(人工标注);OBB: Z(人工标注)                       | 1075²@8-69m(SAR)/10m(光学)     | 1K              | pad              | 1K     |
| 50   | SatlasPretrain                  | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G(多源人工+自动标注+OSM+众包) | 512²@10m                       | 856K            | pad              | 856K   |
| 51   | MADS                            | segmap(LC): C(OSM众包); ; endmember labels(6类: pavement/low vegetation/soil/tree/roof/water): Z(人工标注); ; DSM: G(地理叠加); ; GIS: G(地理叠加) | 1371×888@10m                   | 1场景(1371x888) | pad              | 1      |
| 52   | MAVOC/UNICORN (PBVS)            | class label(10类车辆): Z(PBVS挑战赛标注)                     | EO31²px SAR55²px               | 459K            | pad              | 459K   |
| 53   | SpaceNet 6 MSAW                 | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | RGB: 900²@0.5m; SAR: 900²@0.5m | 3K              | n=2, 2×2=4       | 13K    |

> **备注**：SOMA-1M(#20) 仅测试集3,000对已公开，全量1.3M对在GitHub逐步放出中；SpaceNet 9 不收录——optical与SAR仅提供tie-points（配准本身是其挑战任务），无发布者预配准影像，不符合对齐定义。

### 5. NIR↔MS（30个数据集，总计 198.7M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                        | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ----------------------------- | --------------- | ---------------- | ------ |
| 1    | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m                    | 100K            | n=41, 41×41=1681 | 168.1M |
| 2    | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                     | 4K              | n=5, 5×5=25      | 112K   |
| 3    | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                      | 40K             | n=1              | 40K    |
| 4    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                      | 4K              | n=1              | 4K     |
| 5    | SpaceNet 5             | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                     | 25K             | n=1, 1×1=1       | 25K    |
| 6    | SpaceNet 2             | building polygons: Z(人工标注)                               | 650²@30cm                     | 24K             | n=1              | 24K    |
| 7    | SpaceNet 4             | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                     | 62K             | n=2, 2×2=4       | 248K   |
| 8    | MajorTOM-Core          | cloud mask: P(模型伪标注)                                    | 1068²@10m                     | 2               | n=3, 3×3=9       | 18     |
| 9    | SpaceNet 3             | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified/Cart Track + surface/bridge/lane属性): Z(标注精度≤7px, QA/QC双检); ; metadata(OSM标签体系): Z(人工标注) | 650²@30cm                     | 3K              | n=4, 4×4=16      | 59K    |
| —    | **<512² 数据集**       |                                                              |                               |                 |                  |        |
| 10   | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): Z(QC)(AI预标+人工检查); ; DEM: G(地理叠加); ; land cover: G(地理叠加) | 509²@10m                      | 49K             | pad              | 49K    |
| 11   | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加); ; biomass(ESA CCI): G(地理叠加); ; built-up(GHSL): G(地理叠加); ; veg continuous(MODIS): G(地理叠加); ; DEM(SRTM 90m): G(地理叠加) | 448²@10m                      | 17.0M           | pad              | 17.0M  |
| 12   | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加); ; DEM(Copernicus 30m→10m插值): G(地理叠加); ; NDVI: R(规则生成) | 264²@10m                      | 9.1M            | pad              | 9.1M   |
| 13   | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                      | 180K            | pad              | 180K   |
| 14   | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                      | 285K            | pad              | 285K   |
| 15   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): Z(QC)(自动分类+专家审核+人工精修)      | 256²@10m                      | 20K             | pad              | 20K    |
| 16   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                      | 500             | pad              | 500    |
| 17   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                      | 13K             | pad              | 13K    |
| 18   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                      | 8K              | pad              | 8K     |
| 19   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                      | 6K              | pad              | 6K     |
| 20   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                       | 1K              | pad              | 1K     |
| 21   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                      | 39K             | pad              | 39K    |
| 22   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): Z(QC)(TerraClass专家分类+D | 128²@10m                      | 30K             | pad              | 30K    |
| 23   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m      | 549K            | pad              | 549K   |
| 24   | MMEarth                | DEM: G(地理叠加); ; 冠层高度(ETH-GCHM): G(地理叠加); ; WorldCover: G(ESA,地理叠加); ; Biome/Ecoregion: G(地理叠加); ; ERA5温/降水: G(地理叠加); Dynamic World 9类LandCover: G | 128²@10m                      | 1.2M            | pad              | 1.2M   |
| 25   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                       | 400K            | pad              | 400K   |
| 26   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): Z(QC)(UNet预标+人工修正, ArcGIS Pro); ; DEM+slope: G(地理叠加) | RGB@25cm, S1/S2@10m           | 20K             | pad              | 20K    |
| 27   | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 10²@10m(Sentinel)             | 241K            | n=1              | 241K   |
| 28   | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G(多源人工+自动标注+OSM+众包) | 512²@10m                      | 856K            | pad              | 856K   |
| 29   | MADS                   | segmap(LC): C(OSM众包); ; endmember labels(6类: pavement/low vegetation/soil/tree/roof/water): Z(人工标注); ; DSM: G(地理叠加); ; GIS: G(地理叠加) | 1371×888@10m                  | 1场景(1371x888) | pad              | 1      |
| 30   | SpaceNet 6 MSAW        | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | NIR: 900²@0.5m; MS: 225²@2.0m | 3K              | n=2, 2×2=4       | 13K    |

### 6. NIR↔PAN（6个数据集，总计 681K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                         | 原数量 | 裁剪        | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | ------------------------------ | ------ | ----------- | ------ |
| 1    | SpaceNet 5      | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                      | 25K    | n=1, 1×1=1  | 25K    |
| 2    | SpaceNet 2      | building polygons: Z(人工标注)                               | 650²@30cm                      | 24K    | n=1         | 24K    |
| 3    | SpaceNet 4      | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                      | 62K    | n=2, 2×2=4  | 248K   |
| 4    | SpaceNet 3      | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified/Cart Track + surface/bridge/lane属性): Z(标注精度≤7px, QA/QC双检); ; metadata(OSM标签体系): Z(人工标注) | 650²@30cm                      | 3K     | n=4, 4×4=16 | 59K    |
| 5    | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | NIR: 900²@0.5m; PAN: 900²@0.5m | 3K     | n=2, 2×2=4  | 13K    |
| 6    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | NIR: 512²@0.2m; PAN: 256²@0.4m | 241K   | n=1         | 241K   |

### 7. NIR↔SAR（30个数据集，总计 200.4M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                         | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ------------------------------ | --------------- | ---------------- | ------ |
| 1    | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m                     | 100K            | n=41, 41×41=1681 | 168.1M |
| 2    | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                      | 4K              | n=5, 5×5=25      | 112K   |
| 3    | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                       | 40K             | n=1              | 40K    |
| 4    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                       | 4K              | n=1              | 4K     |
| 5    | C2Seg-AB               | segmap(13类地物): G(OSM叠加+人工检查)                        | 886²@10m                       | 413             | n=2, 2×2=4       | 1K     |
| 6    | SODAS                  | crop-type map(18类+休耕): G(LPIS/SIGPAC欧盟CAP农户申报记录); precipitation(日降雨): G(气象站) | 3587×3607@10m                  | 301(S1)+210(S2) | n=13, 13×13=169  | 50K    |
| 8    | MMM-RS                 | text prompt(4组件信息丰富型,用于文本→图像生成):; ①图像内容描述: P+H(BLIP-2自动生成+人工筛查修正); ②GSD等级(5级:超高/高/普通/低/超低分辨率): R(原GSD按resize比例计算); ③天气(正常/雾/雪/低光): P(部分样本合成); ④卫星类型: Z(源数据集继承) | 512²@0.5-10m                   | 2.1M            | n=1              | 2.1M   |
| —    | **<512² 数据集**       |                                                              |                                |                 |                  |        |
| 9    | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): Z(QC)(AI预标+人工检查); ; DEM: G(地理叠加); ; land cover: G(地理叠加) | 509²@10m                       | 49K             | pad              | 49K    |
| 10   | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加); ; biomass(ESA CCI): G(地理叠加); ; built-up(GHSL): G(地理叠加); ; veg continuous(MODIS): G(地理叠加); ; DEM(SRTM 90m): G(地理叠加) | 448²@10m                       | 17.0M           | pad              | 17.0M  |
| 11   | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加); ; DEM(Copernicus 30m→10m插值): G(地理叠加); ; NDVI: R(规则生成) | 264²@10m                       | 9.1M            | pad              | 9.1M   |
| 12   | GF-FloodNet            | segmap(水体/洪水, 像素级): Z(人工标注)                       | 256²@1.5-4m                    | 13K             | pad              | 13K    |
| 13   | WHU-OPT-SAR            | segmap(7类: 农田/水体/村庄/城市/森林/道路/其他): Z(人工标注) | 256²@5m                        | 100             | pad              | 100    |
| 14   | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                       | 180K            | pad              | 180K   |
| 15   | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                       | 285K            | pad              | 285K   |
| 16   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): Z(QC)(自动分类+专家审核+人工精修)      | 256²@10m                       | 20K             | pad              | 20K    |
| 17   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                       | 500             | pad              | 500    |
| 18   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                       | 13K             | pad              | 13K    |
| 19   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                       | 8K              | pad              | 8K     |
| 20   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                       | 6K              | pad              | 6K     |
| 21   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                        | 1K              | pad              | 1K     |
| 22   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                       | 39K             | pad              | 39K    |
| 23   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): Z(QC)(TerraClass专家分类+D | 128²@10m                       | 30K             | pad              | 30K    |
| 24   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m       | 549K            | pad              | 549K   |
| 25   | MMEarth                | DEM: G(地理叠加); ; 冠层高度(ETH-GCHM): G(地理叠加); ; WorldCover: G(ESA,地理叠加); ; Biome/Ecoregion: G(地理叠加); ; ERA5温/降水: G(地理叠加); Dynamic World 9类LandCover: G | 128²@10m                       | 1.2M            | pad              | 1.2M   |
| 26   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                        | 400K            | pad              | 400K   |
| 27   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): Z(QC)(UNet预标+人工修正, ArcGIS Pro); ; DEM+slope: G(地理叠加) | RGB@25cm, S1/S2@10m            | 20K             | pad              | 20K    |
| 28   | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 10²@10m(Sentinel)              | 241K            | n=1              | 241K   |
| 29   | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G(多源人工+自动标注+OSM+众包) | 512²@10m                       | 856K            | pad              | 856K   |
| 30   | MADS                   | segmap(LC): C(OSM众包); ; endmember labels(6类: pavement/low vegetation/soil/tree/roof/water): Z(人工标注); ; DSM: G(地理叠加); ; GIS: G(地理叠加) | 1371×888@10m                   | 1场景(1371x888) | pad              | 1      |
| 31   | SpaceNet 6 MSAW        | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | NIR: 900²@0.5m; SAR: 900²@0.5m | 3K              | n=2, 2×2=4       | 13K    |

### 8. MS↔PAN（6个数据集，总计 681K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                             | 原数量 | 裁剪        | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | ---------------------------------- | ------ | ----------- | ------ |
| 1    | SpaceNet 5      | road centerline+travel time属性: Z(人工标注);building polygons: Z(人工标注) | 650²@30cm                          | 25K    | n=1, 1×1=1  | 25K    |
| 2    | SpaceNet 2      | building polygons: Z(人工标注)                               | 650²@30cm                          | 24K    | n=1         | 24K    |
| 3    | SpaceNet 4      | building polygons(GeoJSON): Z(CosmiQ Works标注)              | 900²@50cm                          | 62K    | n=2, 2×2=4  | 248K   |
| 4    | SpaceNet 3      | road centerline(7类: Motorway/Primary/Secondary/Tertiary/Residential/Unclassified/Cart Track + surface/bridge/lane属性): Z(标注精度≤7px, QA/QC双检); ; metadata(OSM标签体系): Z(人工标注) | 650²@30cm                          | 3K     | n=4, 4×4=16 | 59K    |
| 5    | SpaceNet 6 MSAW | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | MS: 225²@2.0m; PAN: 900²@0.5m      | 3K     | n=2, 2×2=4  | 13K    |
| 6    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | PAN: 256²@0.4m; MS(SPOT): 64²@1.6m | 241K   | pad         | 241K   |

### 9. MS↔SAR（27个数据集，总计 198.3M）

| 排名 | 数据集                 | 条件                                                         | 原尺寸                        | 原数量          | 裁剪             | 512²后 |
| ---- | ---------------------- | ------------------------------------------------------------ | ----------------------------- | --------------- | ---------------- | ------ |
| 1    | S1S2-Water             | segmap(水体二值掩膜): R+Z(规则生成+人工标注:NDWI阈值+人工验证);DEM(坡度+高程): G(地理叠加) | 10980²@10m                    | 100K            | n=41, 41×41=1681 | 168.1M |
| 2    | AirMOS                 | OBB: Z(人工标注); HBB: D(mask派生:从OBB转换)                 | 1782²@10m                     | 4K              | n=5, 5×5=25      | 112K   |
| 3    | CloudLULC-Set          | segmap(6类: 水体/森林/草地/裸地/建成区/农田, 像素级): Z(人工标注) | 512²@10m                      | 40K             | n=1              | 40K    |
| 4    | Sen1Floods11           | flood/water mask: Z+R(人工标注+规则生成:446人工+4,385算法生成) | 512²@10m                      | 4K              | n=1              | 4K     |
| 5    | MUMUCD                 | segmap(变化检测二值图): G+R(地理叠加+规则生成:从Dynamic World LC自动计算, 无人工标注); ; land cover(Dynamic World 9类): G(地理叠加); ; DEM(SRTM+ASTER 30m): G(地理叠加); 含多类DW LC标签(可扩展至多类CD, 当前仅提供二值变化检测benchmark) | 1536²@10m                     | 70              | n=5, 5×5=25      | 1K     |
| —    | **<512² 数据集**       |                                                              |                               |                 |                  |        |
| 6    | CloudSEN12             | cloud mask(厚云/薄云/云影, 人工标注): Z(QC)(AI预标+人工检查); ; DEM: G(地理叠加); ; land cover: G(地理叠加) | 509²@10m                      | 49K             | pad              | 49K    |
| 8    | M3LEO                  | segmap(ESA WorldCover 11类): G(地理叠加); ; biomass(ESA CCI): G(地理叠加); ; built-up(GHSL): G(地理叠加); ; veg continuous(MODIS): G(地理叠加); ; DEM(SRTM 90m): G(地理叠加) | 448²@10m                      | 17.0M           | pad              | 17.0M  |
| 9    | TerraMesh              | segmap(ESRI全球土地覆盖10m): G(地理叠加); ; DEM(Copernicus 30m→10m插值): G(地理叠加); ; NDVI: R(规则生成) | 264²@10m                      | 9.1M            | pad              | 9.1M   |
| 10   | SEN12MS                | segmap(4层): G(MODIS MCD12Q1全球土地覆盖);metadata(季节/位置CSV): Z(人工标注) | 256²@10m                      | 180K            | pad              | 180K   |
| 11   | OlmoEarth              | segmap(WorldCover 11类): G(地理叠加:ESA WorldCover); CDL作物类型(仅美国): G(地理叠加:USDA); SRTM/OSM/ERA5/冠层高度: G+R | 256²@10m                      | 285K            | pad              | 285K   |
| 12   | LuojiaSET-OSFCR        | segmap(土地覆盖分类): Z(QC)(自动分类+专家审核+人工精修)      | 256²@10m                      | 20K             | pad              | 20K    |
| 13   | HunanMultimodalDataset | segmap(7类LULC): Z(人工标注);DEM: G(地理叠加)                | 256²@10m                      | 500             | pad              | 500    |
| 14   | Plot-Rice v1.0         | segmap(rice/non-rice): P+Z(模型伪标注+人工标注:SAM-2+RF+人工验证) | 256²@10m                      | 13K             | pad              | 13K    |
| 15   | MultiSenGE             | segmap(14类LULC): G(OCSGE2-GEOGRANDEST矢量数据库)            | 256²@10m                      | 8K              | pad              | 8K     |
| 16   | DFC20 / DFC2020        | segmap(10类LULC): G(MODIS MCD12Q1叠加)                       | 256²@10m                      | 6K              | pad              | 6K     |
| 17   | TalloS                 | tree species label(1,000+类属级): G(全球林业数据库Tallo);DEM(高程/坡度/坡向): G(SRTM 30m地理叠加) | 64²@10m                       | 1K              | pad              | 1K     |
| 18   | Sen12Landslides        | segmap(二值: landslide/non-landslide): Z(75,000+滑坡多边形标注); event date metadata: Z(人 | 128²@10m                      | 39K             | pad              | 39K    |
| 19   | CerraData-4MM          | segmap(L1: 7大类 [乔木/牧场/农业/采矿/建筑/水体/其他用途]; L2: 14子类, 层级标签): Z(QC)(TerraClass专家分类+D | 128²@10m                      | 30K             | pad              | 30K    |
| 20   | BigEarthNet            | segmap(19类多标签+像素级参考图): G(地理叠加);metadata(国家/季节/雪/云/阴影): G+R(地理叠加+规则生成:SCL自动产品) | S1:128²@10m, S2:120²@10m      | 549K            | pad              | 549K   |
| 21   | C2Seg-BW               | segmap(13类地物): G(OSM叠加+人工检查)                        | 128²/256²@10m                 | 7K              | pad              | 7K     |
| 22   | MMEarth                | DEM: G(地理叠加); ; 冠层高度(ETH-GCHM): G(地理叠加); ; WorldCover: G(ESA,地理叠加); ; Biome/Ecoregion: G(地理叠加); ; ERA5温/降水: G(地理叠加); Dynamic World 9类LandCover: G | 128²@10m                      | 1.2M            | pad              | 1.2M   |
| 23   | So2Sat LCZ42           | classification label(17类Local Climate Zone): Z(10位专家验证) | 32²@10m                       | 400K            | pad              | 400K   |
| 24   | DeepFlood              | segmap(4类: open water/dry vegetation/inundated vegetation/other): Z(QC)(UNet预标+人工修正, ArcGIS Pro); ; DEM+slope: G(地理叠加) | RGB@25cm, S1/S2@10m           | 20K             | pad              | 20K    |
| 25   | FLAIR-HUB              | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | 10²@10m(Sentinel)             | 241K            | n=1              | 241K   |
| 26   | SatlasPretrain         | segmap+point+polygon+polyline+class+regression+properties(137类7标注类型): Z+G(多源人工+自动标注+OSM+众包) | 512²@10m                      | 856K            | pad              | 856K   |
| 27   | MADS                   | segmap(LC): C(OSM众包); ; endmember labels(6类: pavement/low vegetation/soil/tree/roof/water): Z(人工标注); ; DSM: G(地理叠加); ; GIS: G(地理叠加) | 1371×888@10m                  | 1场景(1371x888) | pad              | 1      |
| 28   | SpaceNet 6 MSAW        | building polygons: G+Z(地籍+QC); building height: G(AHN LiDAR) | MS: 225²@2.0m; SAR: 900²@0.5m | 3K              | n=2, 2×2=4       | 13K    |

### 10. PAN↔SAR（3个数据集，总计 267K）

| 排名 | 数据集          | 条件                                                         | 原尺寸                              | 原数量 | 裁剪       | 512²后 |
| ---- | --------------- | ------------------------------------------------------------ | ----------------------------------- | ------ | ---------- | ------ |
| 1    | SpaceNet 6 MSAW | building polygons: G(地理叠加，荷兰政府地籍数据, 团队人工QC补漏删错); ; building height: G(地理叠加，从AHN LiDAR DSM自动提取匹配至每个polygon) | 900²@0.5m                           | 3K     | n=2, 2×2=4 | 13K    |
| 2    | FLAIR-HUB       | segmap(15类LC+23类作物): Z(人工标注); DSM/20cm级: G(地理叠加); DTM: G(地理叠加) | PAN: 256²@0.4m; SAR(S1): 10²@10.24m | 241K   | pad        | 241K   |