直接在测试类上面添加注解

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = PaymentMain8001.class)
public class MoveSysUserTest {
    @Autowired
    private MoveSysUser moveSysUser;
  
    @Test
    public void testMoveSysUser() {
        boolean res = moveSysUser.moveSysUser();
        System.out.println(res);
    }
}
~~~

@RunWith(SpringRunner.class)  引入spring对junit4的支持.

@SpringBootTest(classes = PaymentMain8001.class 指springboot 的启动类 PaymentMain8001.