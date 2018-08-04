# 使用Spring Cloud Eureka和Spring Cloud openFeign 搭建服务提供者与消费者demo

### GitHub地址: https://github.com/Notcontrol/spring-cloud-demo



# 1、依赖
> Eureka服务端,也就是服务注册中心。同时，它支持高可用，即配置集群形式。

maven依赖
```aidl
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
> Eureka客户端,主要用来向服务注册中心注册自身提供的服务。

maven依赖
```aidl
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
 </dependency>
```
# 2、配置
### Eureka Server 服务注册处的yml配置
```aidl
server:
  port: 1111
eureka:
  instance:
    hostname: localhost
  client:
    # 这里的配置可以实现服务注册中心的高可用，即将自己作为服务向其他服务中心注册自己。这样就形成了一组互相注册的服务注册中心
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    # 注册中心的职责是维护服务实例，不去检索服务，所以关闭 在做服务注册中心高可用时可以将下方两个开启
    fetch-registry: false
    # 不作为客户端注册
    register-with-eureka: false
spring:
  application:
    name: cloud-eureka
```
##### 注:需要在其项目启动类中使用@EnableEurekaServer启动

### Eureka Client server provide 服务提供者的yml配置
```aidl
server:
  port: 2222

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:1111/eureka
spring:
  application:
    name: cloud-client-a
```
##### 注:需要在其项目启动类中使用@EnableEurekaClient启动

### 服务消费者的yml
```aidl
server:
  port: 2222

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:1111/eureka
spring:
  application:
    name: cloud-client-a
```
#### 注:需要在其项目启动类中使用@EnableEurekaClient和@EnableFeignClients 即通过feign形式去消费服务
# 3模拟服务提供者和消费者模型
## 1模拟服务提供者
```aidl
/**
 * 测试服务提供者
 *
 * @author jianglu
 * Created 2018 - 08 - 03 - TIME
 */

@RestController
@RequestMapping("/api/v1")
public class DemoController {


    @GetMapping("/user")
    public ResponseEntity<BaseUser> user(){
        BaseUser baseUser = new BaseUser();
        baseUser.setPassword("123456");
        baseUser.setUsername("张三");
        baseUser.setUserId(1);
        return ResponseEntity.ok(baseUser);
    }
}
```
# 2模拟服务消费者:
先建一个接口，在该接口上使用@FeignClient注解，其中有两个参数，第一个为它所要调用的服务的名称
fallback参数是因为feign整合了Hystrix,所以具有熔断的能力,具体代码如下:
```aidl
@FeignClient(value = "cloud-client-a",fallback = SchedualServiceHiHystrix.class)
public interface FindService {

    @RequestMapping("/api/v1/user/")
    ResponseEntity<BaseUser> getBaseUser();
}
```
实现了FindService接口的断路器,如果其所调用的服务出现了故障，将返回由下方方法定义的信息
```aidl
/**
 * 断路器
 *
 * @author jianglu
 * Created 2018 - 08 - 04 - TIME
 */
@Component
public class SchedualServiceHiHystrix implements FindService {
    @Override
    public ResponseEntity<BaseUser> getBaseUser() {

        BaseUser baseUser = new BaseUser();
        baseUser.setUserId(1);
        baseUser.setUsername("wangxiaowu");
        baseUser.setPassword("wang");
        return ResponseEntity.ok(baseUser);
    }
}
```
在controller层，暴露一个/index的api接口，并且通过上方定义的Feign客户端FindService来消费服务:
```aidl
/**
 * 服务b的demo
 *
 * @author jianglu
 * Created 2018 - 08 - 03 - TIME
 */
@RestController
public class DemoController {

    @Resource
    FindService findService;


    @GetMapping("/index")
    public ResponseEntity<BaseUser> getUserByB(){
        return findService.getBaseUser();
    }

}

```
# 本文参考资料:
https://blog.csdn.net/forezp/article/details/81040965