* **常用空间查询函数**
ST_AsText(geography) returns text
ST_GeographyFromText(text) returns geography
ST_AsBinary(geography) returns bytea
ST_GeogFromWKB(bytea) returns geography
ST_AsSVG(geography) returns text
ST_AsGML(geography) returns text
ST_AsKML(geography) returns text
ST_AsGeoJson(geography) returns text
ST_Distance(geography, geography) returns double
ST_DWithin(geography, geography, float8) returns boolean
ST_Area(geography) returns double
ST_Length(geography) returns double
ST_Covers(geography, geography) returns boolean
ST_CoveredBy(geography, geography) returns boolean
ST_Intersects(geography, geography) returns boolean
ST_Buffer(geography, float8) returns geography[1]
ST_Intersection(geography, geography) returns geography[1]



* **查询坐标系**

~~~sql
select st_srid(wkb_geometry) from test_file_db_0818001
~~~

* 创建空间索引

  ~~~sql
  create index idx_t_pos_1 on t_pos using gist(pos);
  ~~~

* 计算面的周长

  ~~~sql
  SELECT 
      ST_Perimeter(geom::geography) AS perimeter_geog
  FROM data;
  ~~~

  
