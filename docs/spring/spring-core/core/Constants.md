# Constants

该类用来获取类中的常量

```java
package org.springframework.core;

import java.lang.reflect.Field;
import java.util.*;

import org.springframework.util.Assert;
import org.springframework.util.ReflectionUtils;

/**
 * @since 16.03.2003
 *
 * 根据常量名获取值
 * 根据常量名 前缀/后缀 获取 常量名/值 列表
 * 等
 */
public class Constants {
    
    ...

	public Constants(Class<?> clazz) {
		Assert.notNull(clazz, "Class must not be null");
		this.className = clazz.getName();
		Field[] fields = clazz.getFields();
		for (Field field : fields) {
            // 字段是否由 public static final 修饰 
			if (ReflectionUtils.isPublicStaticFinal(field)) {
				String name = field.getName();
				try {
					Object value = field.get(null);
                    // 缓存字段名和字段值
					this.fieldCache.put(name, value);
				}
				catch (IllegalAccessException ex) {
					// just leave this field and continue
				}
			}
		}
	}

	...

	// 常量 名和值 的映射
	protected final Map<String, Object> getFieldCache() { ... }

	public Number asNumber(String code) throws ConstantException { ... }

	public String asString(String code) throws ConstantException { ... }

	public Object asObject(String code) throws ConstantException { ... }

   	...

	/**
	 * 驼峰式属性名转换为常量名
	 * 
	 * Example: "imageSize" -> "IMAGE_SIZE"
	 * Example: "imagesize" -> "IMAGESIZE"
	 * Example: "ImageSize" -> "_IMAGE_SIZE"
	 * Example: "IMAGESIZE" -> "_I_M_A_G_E_S_I_Z_E"
	 */
	public String propertyToConstantNamePrefix(String propertyName) {
		StringBuilder parsedPrefix = new StringBuilder();
		for (int i = 0; i < propertyName.length(); i++) {
			char c = propertyName.charAt(i);
			if (Character.isUpperCase(c)) {
				parsedPrefix.append("_");
				parsedPrefix.append(c);
			}
			else {
				parsedPrefix.append(Character.toUpperCase(c));
			}
		}
		return parsedPrefix.toString();
	}

}

```

## 示例

```java
package xyz.kail.blog.spring;

import org.springframework.core.Constants;

public class Main {

    public static final String ABC = "123";

    public static void main(String[] args) {

        Constants constants = new Constants(Main.class);
        // 123
        System.out.println(constants.asString("ABC"));
        // [ABC]
        System.out.println(constants.getNames("A"));
        // [123]
        System.out.println(constants.getValues("A"));
        // [123]
        System.out.println(constants.getValuesForSuffix("C"));

    }
}

```

