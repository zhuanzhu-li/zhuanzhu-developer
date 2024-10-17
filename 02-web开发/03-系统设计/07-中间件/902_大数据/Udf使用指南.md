[toc]

# 简介

## 常用类

### 1. GenericUDAFResolver2

参考：https://gist.github.com/developer-sdk/4f33cd5444989d63ef955e33e661d5be
		   https://blog.csdn.net/weixin_33893473/article/details/87983926

*  **示例代码**

~~~java
public class SumInt extends AbstractGenericUDAFResolver {

	@Override
	public GenericUDAFEvaluator getEvaluator(TypeInfo[] info) throws SemanticException {
		 
		if (info.length != 1) {
			throw new UDFArgumentTypeException(info.length - 1, "Exactly one argument is expected.");
		}
		
		if (info[0].getCategory() != ObjectInspector.Category.PRIMITIVE) {
			throw new UDFArgumentTypeException(0, "Only primitive type arguments are accepted but " + info[0].getTypeName() + " was passed as parameter 1.");
		}
        
		if (((PrimitiveTypeInfo)info[0]).getPrimitiveCategory() == PrimitiveCategory.STRING) {
			return new SumStringEvaluator();
		} else if (((PrimitiveTypeInfo)info[0]).getPrimitiveCategory() == PrimitiveCategory.INT) {
			return new SumIntEvaluator();
		} else {
			throw new UDFArgumentTypeException(0, "Only string, int type arguments are accepted but " + info[0].getTypeName() + " was passed as parameter 1.");
		}
	}
	
	
	/**
	 * 
	 * @author User
	 *
	 */
	public static class SumStringEvaluator extends GenericUDAFEvaluator {

		private PrimitiveObjectInspector inputOI;
		
		static class SumAggregationBuffer implements AggregationBuffer {
			int sum;
		}
		
		@Override
		public ObjectInspector init(Mode m, ObjectInspector[] parameters) throws HiveException {
			super.init(m, parameters);
			
			inputOI = (PrimitiveObjectInspector) parameters[0];
			return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
		}
		
		@Override
		public AggregationBuffer getNewAggregationBuffer() throws HiveException {
			SumAggregationBuffer sum = new SumAggregationBuffer();
			reset(sum);
			return sum;
		}

		@Override
		public void reset(AggregationBuffer agg) throws HiveException {
			((SumAggregationBuffer) agg).sum = 0;
		}

		@Override
		public void iterate(AggregationBuffer agg, Object[] parameters) throws HiveException {	
			if(parameters.length != 0 && inputOI.getPrimitiveJavaObject(parameters[0]) != null) {
				((SumAggregationBuffer) agg).sum += Integer.parseInt(inputOI.getPrimitiveJavaObject(parameters[0]).toString());
			}
		}

		@Override
		public Object terminatePartial(AggregationBuffer agg) throws HiveException {
			return ((SumAggregationBuffer) agg).sum;
		}

		@Override
		public void merge(AggregationBuffer agg, Object partial) throws HiveException {
			((SumAggregationBuffer) agg).sum += Integer.parseInt(inputOI.getPrimitiveJavaObject(partial).toString());
		}

		@Override
		public Object terminate(AggregationBuffer agg) throws HiveException {
			return ((SumAggregationBuffer) agg).sum;
		}
		
	}
	
	/**
	 * 
	 * @author User
	 *
	 */
	public static class SumIntEvaluator extends GenericUDAFEvaluator {

		private IntObjectInspector inputOI;
		
		static class SumAggregationBuffer implements AggregationBuffer {
			int sum;
		}
		
		@Override
		public ObjectInspector init(Mode m, ObjectInspector[] parameters) throws HiveException {
			super.init(m, parameters);
			
			inputOI = (IntObjectInspector) parameters[0];
			return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
		}
		
		@Override
		public AggregationBuffer getNewAggregationBuffer() throws HiveException {
			SumAggregationBuffer sum = new SumAggregationBuffer();
			reset(sum);
			return sum;
		}

		@Override
		public void reset(AggregationBuffer agg) throws HiveException {
			((SumAggregationBuffer) agg).sum = 0;
		}

		@Override
		public void iterate(AggregationBuffer agg, Object[] parameters) throws HiveException {
			((SumAggregationBuffer) agg).sum += inputOI.get(parameters[0]);
		}

		@Override
		public Object terminatePartial(AggregationBuffer agg) throws HiveException {
			return ((SumAggregationBuffer) agg).sum;
		}

		@Override
		public void merge(AggregationBuffer agg, Object partial) throws HiveException {
			((SumAggregationBuffer) agg).sum += inputOI.get(partial);
		}

		@Override
		public Object terminate(AggregationBuffer agg) throws HiveException {
			return ((SumAggregationBuffer) agg).sum;
		}
		
	}
}
~~~

* **点数据聚合**

实现根据点设置缓冲器（buffer）进行面聚合, 使用spark-beeline执行时，发现执行部分静态方法时，报错详细不明显，建议执行时，查看sparkSQL的日志文件进行分析。

~~~java
package com.dtsw.gis;

import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.hive.serde2.io.DoubleWritable;
import org.geotools.geometry.jts.JTS;
import org.geotools.referencing.CRS;
import org.locationtech.jts.geom.Coordinate;
import org.locationtech.jts.geom.Geometry;
import org.locationtech.jts.geom.GeometryFactory;
import org.locationtech.jts.io.WKBWriter;
import org.opengis.referencing.crs.CoordinateReferenceSystem;

import java.util.ArrayList;
import java.util.List;

/**
 * @author liwenqi
 */
public class MultiPointBufferUnionUDF
        extends UDF {

    public String evaluate(DoubleWritable radius, ArrayList<String> lonLats) {
        try {
            Geometry geometry = null;
            for (String lonLat : lonLats) {
                String[] lonlatArr = simpleSplitStr(lonLat, ",");
                if (lonlatArr == null || lonlatArr.length != 2) {
                    System.out.println("ERROR data !");
                    continue;
                }
                double lon = Double.parseDouble(lonlatArr[0]);
                double lat = Double.parseDouble(lonlatArr[1]);
                double[] lonLat3857 = new double[2];
                //地球半径
                double earthRad = 6378137.0;
                lonLat3857[0] = lon * Math.PI / 180 * earthRad;
                double param = lat * Math.PI / 180;
                lonLat3857[1] = earthRad / 2 * Math.log((1.0 + Math.sin(param)) / (1.0 - Math.sin(param)));

                geometry = geometry == null ? pointBuffer(lonLat3857[0], lonLat3857[1], radius.get()) : geometry.union(pointBuffer(lonLat3857[0], lonLat3857[1], radius.get()));
            }
            CoordinateReferenceSystem crs4326 = CRS.parseWKT("GEOGCS[\"WGS 84\",DATUM[\"WGS_1984\",SPHEROID[\"WGS 84\",6378137,298.257223563,AUTHORITY[\"EPSG\",\"7030\"]],AUTHORITY[\"EPSG\",\"6326\"]],PRIMEM[\"Greenwich\",0,AUTHORITY[\"EPSG\",\"8901\"]],UNIT[\"degree\",0.0174532925199433,AUTHORITY[\"EPSG\",\"9122\"]],AUTHORITY[\"EPSG\",\"4326\"]]");
            CoordinateReferenceSystem crs3857 = CRS.parseWKT("PROJCS[\"WGS 84 / Pseudo-Mercator\", \n" +
                    "  GEOGCS[\"WGS 84\", \n" +
                    "    DATUM[\"World Geodetic System 1984\", \n" +
                    "      SPHEROID[\"WGS 84\", 6378137.0, 298.257223563, AUTHORITY[\"EPSG\",\"7030\"]], \n" +
                    "      AUTHORITY[\"EPSG\",\"6326\"]], \n" +
                    "    PRIMEM[\"Greenwich\", 0.0, AUTHORITY[\"EPSG\",\"8901\"]], \n" +
                    "    UNIT[\"degree\", 0.017453292519943295], \n" +
                    "    AXIS[\"Geodetic longitude\", EAST], \n" +
                    "    AXIS[\"Geodetic latitude\", NORTH], \n" +
                    "    AUTHORITY[\"EPSG\",\"4326\"]], \n" +
                    "  PROJECTION[\"Popular Visualisation Pseudo Mercator\", AUTHORITY[\"EPSG\",\"1024\"]], \n" +
                    "  PARAMETER[\"semi_minor\", 6378137.0], \n" +
                    "  PARAMETER[\"latitude_of_origin\", 0.0], \n" +
                    "  PARAMETER[\"central_meridian\", 0.0], \n" +
                    "  PARAMETER[\"scale_factor\", 1.0], \n" +
                    "  PARAMETER[\"false_easting\", 0.0], \n" +
                    "  PARAMETER[\"false_northing\", 0.0], \n" +
                    "  UNIT[\"m\", 1.0], \n" +
                    "  AXIS[\"Easting\", EAST], \n" +
                    "  AXIS[\"Northing\", NORTH], \n" +
                    "  AUTHORITY[\"EPSG\",\"3857\"]]");
            return WKBWriter.toHex(new WKBWriter().write(JTS.transform(geometry, CRS.findMathTransform(crs3857, crs4326))));
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    private static String[] simpleSplitStr(String str, String splitStr) {
        if (str == null || "".equals(str.trim())) {
            return null;
        }
        List<String> strings = new ArrayList<>();
        while (str.contains(splitStr) || str.length() > 0) {
            int index = str.indexOf(splitStr);
            if (index == -1) {
                strings.add(str);
                break;
            } else {
                strings.add(str.substring(0, index));
            }
            str = str.substring(index + 1);
        }
        return strings.toArray(new String[0]);
    }

    public static Geometry pointBuffer(Double lon, Double lat, Double radius) {

        return new GeometryFactory().createPoint(new Coordinate(lon, lat)).buffer(radius);
    }
}

~~~





* **hive使用**
* **SparkSql使用**

使用sparkSQL,会存在缓存问题，在使用时注意修改包名，或者重启SqarkSQL服务

~~~sql
create temporary function MultiPointBufferUnionUDF as 'com.dtsw.gis.MultiPointBufferUnionUDF' using jar '/home/mr/gs-utils-udf-1.0.0.jar';

select MultiPointBufferUnionUDF(double(5000),array("104.06659,30.65429", "104.10229,30.62901", "103.4741,30.2092", "105.3307,29.9191"));
~~~



