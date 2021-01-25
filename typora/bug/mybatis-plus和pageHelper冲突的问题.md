## 后端使用mybatis-plus实现数据的查询,本地能正常的执行,但是部署到项目会报异常Caused by: org.apache.ibatis.builder.BuilderException: Error evaluating expression 'ew.customSqlSegment'. Cause: org.apache.ibatis.ognl.OgnlException: customSqlSegment [com.baomidou.mybatisplus.core.exceptions.MybatisPlusException: This is impossible to happen]

​	**先说解决的办法:**将mybatis-plus升级到最新版本:

```xml
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <!-- <version>3.2.0</version>-->
            <version>3.4.2</version>
        </dependency>
```

​	如果没能解决你的问题,你也可以继续读一读,可以参考我的解决思路.

​	**问题描述:使用mybatisplus查询数据,本地正常运行,部署到项目的时候,会报错以上异常,所有类似以下的地方都报错:**

```sql
@Select("SELECT count(src_ip) as attackSourceNum,src_ip as ip,src_nation as srcNation FROM protect_event ${ew.customSqlSegment} order by attackSourceNum desc")
List<VideoAttackSourceResponseVO> selectVideoAttackSource(@Param(Constants.WRAPPER) QueryWrapper<ProtectEvent> queryWrapper);
```

​	报错原因 ${ew.customSqlSegment}转义不过来.经过远程debug(自行百度,我也不怎么会),发现走到该方法的时候,进入的**其它版本的mybatis-spring-boot-start**,所以怀疑是jar包冲突(其实一开始就应该先看看是不是jar包冲突的),经过maven-helper插件分析(也可以用mvn tree 相关的命令),**引起冲突的是pageHelper**.**一共有mybatis和mybatis-spring两处冲突:**

![image-20210124203828083](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20210124203828083.png)

![image-20210124203919175](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20210124203919175.png)

​	因为是想让mybatis-plus生效,所以就想把pageHelper中的冲突的依赖去掉,但是当我们把mybatis-spring-boot-starter去除依赖后,项目启动就会报错.所以只好手动的(而非用maven-helper插件)**在pageHelper中直接去掉mybatis和mybatis-spring的依赖,为了保险起见,还将mybatis-plus升级到最新的版本**.经过这样的处理,打包到线上,发现数据都查询了出来,问题完美的解决,后来又验证了一下,发现将mybatis-plus升级到最新的版本,而不用管pageHelper的依赖,mybatis-spring的冲突没有了,但是mybatis的冲突还是在,但即使这样还是能查询出来数据.所以怀疑这个异常的问题是**mybatis-spring的jar包冲突所致.**

​	以上就是我的解决问题的过程,其实还可以在pom.xml中直接声明mybatis和mybatis-spring的版本,maven采用的声明优先原则,所以会走你自己声明的配置.另外如果依赖路径一样长,谁在pom.xml上先声明(即谁在上面),谁依赖的版本有效,有时候可以试试将自己想要生效的jar包往上移动一下.