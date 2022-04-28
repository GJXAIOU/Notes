# Java 日期类型对象通过 MyBatis 向数据库中的存取

## 一、数据库中的日期数据类型

详见[官方文档](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-types.html)

数据库中的日期数据类型有四种：date、datetime、timestamp、time。

- date 类型只保存年月日，不保存时分秒

- datetime 和 Timestamp 保存年月日时分秒

- time只保存时分秒。数据库字段值进行比较时

date只比较年月日，datetime和Timestamp比较年月日时分秒，time只比较 时分秒。
datetime、Timestamp 在数据库中的存储结构不一样，Timestamp更节省空间，但对于 java 对象的存取都是一样的。

## 二、java中的四种日期类型：

Java 中的四种日期类型为：`java.util.Date`、`java.sql.Date`、`java.sql.Timestamp`、`java.sql.Time`。

- `java.sql.date`、`java.sql.Timestamp`、` java.sql.Time` 都是 `java.util.Date`类的子类（即都继承了 `java.util.Date`）。

    `java.sql` 包下的 date 显示年月日， Timestamp 显示年月日时分秒，Time 显示时分秒，`java.sql.date` 显示年月日时分秒。

- 这四种日期类型内部保存的都是时间戳。虽然 `java.sql.Date`、`java.sql.Time` 值显示年月日或时分秒，但其内部保存的都是完整的时间戳，因此当把 `java.sql.Date`、`java.sql.Time` 转型为 `java.uitl.Date` 类型时，还是能显示完整的年月日时分秒。

- 直接 `System.out.println(java.sql.date)` 显示年月日，`System.out.println(java.sql.time)` 显示时分秒。但是通过

    ```java
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String str = simpleDateFormat.format(objDate)
    ```


    无论 objDate 是四种类型中的任何类型，返回的 str 字符串都是准确的完整的年月日时分秒。因`simpleDateFormat.format()` 方法的参数是类型时 `java.util.Date` 类型，`java.sql.date`、`Timestamp`、`time` 会自动转为 `java.util.Date` 类型。

### 三、数据库中的四种日期数据类型通过 mybatis 逆向工程生成的实体类对应属性的类型都是 java.util.Date（不是java.sql.Date）类型。

### java.util.Date 类型的 java 数据保存与读取

- `java.util.Date` 类型的 java 数据（如 `2019-05-08 14:56:23`）向数据库 `Date` 类型字段存值时，会忽略掉 Date java对象中的时分秒 ，只保存年月日(2019-05-08)，当再从数据库中读取 date 类型字段值赋值为 java 的 Date 对象时，Date 对象的值为 `2019-05-08 00:00:00`，时分秒变成 `00：00：00`，保存到数据库之前的原有的时分秒（14:56:23）丢失。
- `java.util.Date` 类型的 java 数据（如 `2019-05-08 14:56:23`）向数据库 `datetime`、`Timestamp` 类型字段存值时，其年月日时分秒都会保存，再从数据库中取值赋给 Date 对象，还是 `2019-05-08 14:56:23`。
- `java.util.Date`类型的 java 数据（如 `2019-05-08 14:56:23`）向数据库 `Time` 类型字段存值时，会忽略掉年月日，只保存时分秒 ，当再从数据库中读取 time 类型字段值赋值为 java 的 Date 对象时，Date 对象的值为 `1970-01-01 14:56:23`，保存到数据库之前的原有的年月日 信息（2019-05-08）丢失。 　　

### java.sql.Date 类型的 java 数据保存

- `java.sql.Date` 类型的 java 数据向数据库 Date 类型字段存值时，因 `java.sql.Date` 只会显示年月日不显示时分秒，因此只保存年月日(如 `2019-05-08`)，当再从数据库中读取 Date 类型字段值赋值为 java 的 Date 对象时，Date 对象的值为 `2019-05-08`。
- `java.sql.Date` 类型的 java 数据向数据库 Datetime、Timestamp类型字段存值时，因java.sql.Date只会显示年月日不显示时分 秒（即使对象内部保存的是时间戳，有时分秒信息），保存到数据库datetime、Timestamp类型字段的值为2019-05-08 00：00：00，再从数 据库中取值赋给Date（无论是java.util.Date还是java.sql.Date）对象，值为2019-05-08 00：00：00
- `java.sql.Date` 类型的 java 数据向数据库time类型字段存值时，因java.sql.Date只会显示年月日不显示时分秒（即使对象内部 保存的是时间戳，有时分秒信息），因此会报错，存储失败。

### java.sql.Time 类型的 java 数据保存

- 因 time 类型只显示时分秒如：`12:12:12`，因此向数据库 time 字段保存时，只保存时分秒，当在从数据库取值赋值给 Time 对象， Time 对象如果转为 `java.util.Date` 对象，其年月日信息丢失，显示为 `1970-01-01 12:12:12`；
- 因 time 类型只显示时分秒，当向数据库 date、datetime、Timestamp 字段存值时会报错。

注：`java.sql.Timestimp` 类型的 java 数据保存和 `java.util.Date` 的保存情况一样

当从数据库中读取数据存入 java 对象时。`java.util.Date` 存储读取的年月日时分秒，`java.sql.Date` 只存储读取的年月日，即使读取的数据库 字段是 `datetime` 字段，有时分秒，如果用 `java.sql.Date` 接收读取的数据，还是值存年月日，即使 `java.sql.Date` 转成 `java.util.Date` 对象， 时分秒的值还是 `00：00：00`。Time 对象只存储数据库从读取的时分秒值，年月日为 `1970-01-01`。

### 四、MyBatis 的 Mapper 配置文件中的 jdbcType

jdbcType 相当于是一个数据拦截器，当向数据库存取时，jdbcType 在入库之前起作用，当从获取库取数据时，jdbcType 在从数据库取出数值时候和向 java 对象赋值之前起拦截作用。　

```xml
<resultMap id="BaseResultMap" type="test.entity.Datetest" >
    <id column="id" property="id" jdbcType="INTEGER" />
    <result column="datestimp" property="datestimp" jdbcType="TIMESTAMP" />
    <result column="datetest" property="datetest" jdbcType="DATE" />
    <result column="time" property="time" jdbcType="TIME" />
</resultMap>

<insert id="insert" parameterType="test.entity.Datetest" >
    insert into datetest (id, datestimp, datetest, time)
    values (#{id,jdbcType=INTEGER}, #{datestimp,jdbcType=TIMESTAMP}, #{datetest,jdbcType=DATE}, #{time,jdbcType=TIME})
</insert>
```

MyBatis 逆向工程生产的 mapper 文件的 jdbcType 的设置为：

- date 类型字段对应的 jdbcType 类型为 `jdbcType="DATE"`
- datetime 和 Timestamp 类型字段对应的为 `jdbcType="TIMESTAMP"`
- time 字段对应个为 `jdbcType="TIME"`
    　　

注意：**只有当 java 对象为 `java.util.Date` 类型时，jdbcType 对数据的存取才起作用，对其他的 java 时间对象，jdbcType 没有作用。** 　

jdbcType 类型对 `java.util.Date`日期数据向数据库存取的影响：

- 当 `jdbcType="DATE"`，当向数据库存取数据时会过滤掉时分秒。无论数据库中的字段是 `date`、`datetime`、`Timestamp` 中的哪一种，最终保存的都只有年月日，如果字段类型为 `datetime`、 `Timestamp`，时分秒信息为 `00:00:00`。如果 java 对象类型为 time 或者数据库字段类型为 time 类型，会报错。

    当从数据库中取数据时，无论数据字段类型为 `date`、`datetime`、`Timestamp`、`time` 哪一种，最终取到的只有年月日，时分秒为 `00:00:00`。从 time 字段取值存到 java 的 date 对象显示为 `1970-01-01 00:00:00`。

- 当 `jdbcType="TIMESTAMP"` 时，jdbcType 不过滤任何内容，对存取没影响。

- 当 `jdbcType="TIME"` 时，当向数据库存取数据时会过滤掉年月日。向数据库中存数据，如果数据库字段为`date`、`datetime`、`Timestamp`会报错。从数据库取数据存入 java 的 date 对象时，年月日都会变成 `1970-01-01`

可以不设置 jdbcType 时，不设置 jdbcType 相当于少了一层数据拦截。

