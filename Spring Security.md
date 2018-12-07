## Security

### Security Plugin

#### SecurityContextHolder

该类主要用于存储Security Context

提供四种策略

| 策略名                      | 策略实现                                            |
| --------------------------- | --------------------------------------------------- |
| MODE_THREADLOCAL(默认)      | ThreadLocalSecurityContextHolderStrategy            |
| MODE_INHERITABLETHREADLOCAL | InheritableThreadLocalSecurityContextHolderStrategy |
| MODE_GLOBAL                 | GlobalSecurityContextHolderStrategy                 |



```java
public class SecurityContextHolder {
	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
	public static final String MODE_GLOBAL = "MODE_GLOBAL";
	public static final String SYSTEM_PROPERTY = "spring.security.strategy";
    //可以通过启动命令指定策略
	private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
	private static SecurityContextHolderStrategy strategy;
	private static int initializeCount = 0;

    //类加载时初始化
	static {
		initialize();
	}
    
	private static void initialize() {
		if (!StringUtils.hasText(strategyName)) {
            //默认使用ThreadLocal
			strategyName = MODE_THREADLOCAL;
		}

		if (strategyName.equals(MODE_THREADLOCAL)) {
			strategy = new ThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
			strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_GLOBAL)) {
			strategy = new GlobalSecurityContextHolderStrategy();
		}
		else {
			try {
				Class<?> clazz = Class.forName(strategyName);
				Constructor<?> customStrategy = clazz.getConstructor();
				strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
			}
			catch (Exception ex) {
				ReflectionUtils.handleReflectionException(ex);
			}
		}
		initializeCount++;
	}
}
```



#### Security Context

用于获取用户的信息，存储在成员变量`Authentication`中

```java
public class SecurityContextImpl implements SecurityContext {
	//存储用户信息
	private Authentication authentication;
}
```

