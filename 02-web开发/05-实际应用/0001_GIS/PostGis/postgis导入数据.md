## 导入csv文件

* 先使用csv正常导入功能导入数据
* 执行如下sql
  
        -- 创建 geom字段
        SELECT AddGeometryColumn ('南昌带高度建筑20米栅格数据（带geom)', 'geom', 4326, 'MULTISURFACE', 2);
        -- 修改 geom字段为通用类型
        ALTER TABLE "南昌带高度建筑20米栅格数据（带geom)" ALTER COLUMN geom SET DATA TYPE geometry;
        -- 将原有geom字段（varchar） 转为 geom字段的值
        update "南昌带高度建筑20米栅格数据（带geom)" set geom =  (SELECT ST_SetSRID( geom1::geometry,4326) AS wkb_geometry);
    
* 使用qgis 转为需要的格式即可

* 修改图层坐标系

``` 
SELECT UpdateGeometrySRID('ny_union_test001','geom',4326);
```