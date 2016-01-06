title: "hive udf使用"
date: 2015-09-24 18:21:35
tags: hive
---

hive以udf的方式提供了扩展hive的方法。每个udf只需要继承hive指定的类，通过固定的方法名识别出提供的方法。工作中经常遇到这种场景：计算周累计或者月累计。计算的逻辑是判断今天与昨天是否属于同一个周（月），如果是，结果为昨天结果加上今天的结果，否则结果为今天的结果。用hql如下：
```sql
CASE WHEN WEEKOFYEAR('${TIME}')= WEEKOFYEAR('${BEFORETIME}') AND F1.LOGIN_YN = 'Y' THEN
    COALESCE(F2.WK_DAY_CNT,0) + 1
WHEN WEEKOFYEAR('${TIME}')= WEEKOFYEAR('${BEFORETIME}') THEN COALESCE(F2.WK_DAY_CNT,0) 
    WHEN F1.LOGIN_YN = 'Y' THEN 1 ELSE 0 END)
```
现在我想把这段代码用一个udf实现:
```
    week_acc([time], [beforeTime], value, accValue)
```
---
现在开始动手实现：
1. 新建maven工程，pom文件添加hive的开发工具包以来以及joda依赖：
```
<dependency>
      <groupId>org.apache.hive</groupId>
      <artifactId>hive-pdk</artifactId>
      <version>0.10.0</version>
    </dependency>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-core</artifactId>
      <version>1.2.1</version>
    </dependency>
    <dependency>
      <groupId>joda-time</groupId>
      <artifactId>joda-time</artifactId>
      <version>2.8.2</version>
    </dependency>
```
2. 编写自定义的udf类，继承UDF类
```
public abstract class WeekAccumulator extends UDF {

    public LongWritable evaluate(Text timeStr,
                                 Text beforeTimeStr,
                                 LongWritable todayValue,
                                 LongWritable beforeAccumulator) {
        return inSamePeriod(timeStr, beforeTimeStr) ?
                new LongWritable(todayValue.get() + beforeAccumulator.get()) : todayValue;
    }

    public LongWritable evaluate(LongWritable todayValue, LongWritable beforeAccumulator) {
        DateTime dateTime = new DateTime();
        Text timeStr = new Text(dateTime.toString("yyyy-MM-dd"));
        Text beforeTimeStr = new Text(dateTime.minusDays(1).toString("yyyy-MM-dd"));
        return evaluate(timeStr, beforeTimeStr, todayValue, beforeAccumulator);
    }

    private boolean inSamePeriod(Text timeStr, Text beforeTimeStr) {
        DateTime thisDate = new DateTime(timeStr.toString());
        DateTime beforeDate = new DateTime(beforeTimeStr.toString());
        return thisDate.getWeekOfWeekyear() == beforeDate.getWeekOfWeekyear();
    }
}
```
3.打包（注意要把依赖的jar包也打进jar包里，否则运行时会找不到依赖的jar包。）
4. 将jar包copy到hdfs中
5. 编辑hive conf文件夹下的.hiverc文件(没有手动创建),文件内容：
```
add jar hdfs://{MASTER}/{udf path}
create temporary function week_acc as '{class}';
```
---

**打开hive cli，输入:**
```select week_acc('2015-09-14', '2015-09-13', 1, 10)```
**执行成功，那么udf就完成了。**

## 注意事项 ##
如果hive设置hive.exec.mode.local.auto（默认false）为true的话，上面命令会执行失败，提示jar文件找不到。只能把这个配置改成false了。
