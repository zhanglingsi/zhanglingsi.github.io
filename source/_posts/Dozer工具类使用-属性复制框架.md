---
title: Dozer工具类使用-属性复制框架
categories: 编程
copyright: true
---

## Dozer工具类使用-属性复制框架

## Quick Start

### Dozer配置

[Maven]
```bash
<dependency>
    <groupId>net.sf.dozer</groupId>
    <artifactId>dozer</artifactId>
    <version>5.4.0</version>
</dependency>
```

[官方文档] (http://dozer.sourceforge.net/documentation/about.html)

<!-- more -->

### 第一个示例  Map转换JavaBean
假设由request得到了页面参数map，想将其转为领域对象–Product。
Product有如下几个属性：
private Long id;
private String name;
private String description;
``` bash
Map<String,Object> map = Maps.newHashMap();
map.put("id", 10001L);
map.put("name", "张三");
map.put("description", "吉祥如意");

DozerBeanMapper mapper = new DozerBeanMapper();
Product product = mapper.map(map, Product.class);
assertThat(product.getId()).isEqualTo(10001);
assertThat(product.getName()).isEqualTo("张三");
assertThat(product.getDescription()).isEqualTo("吉祥如意");
```

### 第二个示例  JavaBean转换DTO
        
如下定义了一个DTO对象，但属性名并不匹配Product对象中的属性名，如下所示：

```bash
private long productId;
private String productName;
private String desc;
```
这时简单的在相关属性上加一个注解即可，如下所示：
```bash
@Mapping("id")
private long productId;

@Mapping("name")
private String productName;
@Mapping("description")
private String desc;

完整代码示例见：

Product product = new Product();
product.setId(10201L);
product.setName("Ethen");
product.setDescription("吉祥如意");
DozerBeanMapper mapper = new DozerBeanMapper();
ProductDTO productDTO = mapper.map(product, ProductDTO.class);
assertThat(productDTO.getProductId()).isEqualTo(10201L);
assertThat(productDTO.getProductName()).isEqualTo("Ethen");
assertThat(productDTO.getDesc()).isEqualTo("吉祥如意");
```

附：工具类完整代码
```bash
/**
 * <ul>
 * <li>简单封装Dozer, 实现深度转换Bean<->Bean的Mapper.实现:</li>
 * <li>1、持有Mapper的单例.</li>
 * <li>2、泛型返回值转换.</li>
 * <li>3、批量转换Collection中的所有对象.</li>
 * <li>4、区分创建新的B对象与将对象A值复制到已存在的B对象两种函数.</li>
 * </ul>
 */
public class BeanCopier {

	/**
	 * 持有Dozer单例, 避免重复创建DozerMapper消耗资源.
	 */
	private static DozerBeanMapper dozer = new DozerBeanMapper();

	/**
	 * 基于Dozer转换对象的类型.
	 */
	public static <T> T copy(Object source, Class<T> destinationClass) {
        if(source == null){
            return null;
        }
		return dozer.map(source, destinationClass);
	}

	/**
	 * 基于Dozer将对象A的值拷贝到对象B中.
	 */
	public static void copy(Object source, Object destinationObject) {
        if(source != null)
        {
		dozer.map(source, destinationObject);
        }
	}

    public static <T> List<T> mapList(Collection sourceList, Class<T> destinationClass) {
        List destinationList = Lists.newArrayList();
        for (Iterator i$ = sourceList.iterator(); i$.hasNext(); ) { Object sourceObject = i$.next();
            Object destinationObject = dozer.map(sourceObject, destinationClass);
            destinationList.add(destinationObject);
        }
        return destinationList;
    }

    /**
     * 将源集合转换为目标集合,注意:目标集合是新建的
     * @param <T>
     * @param srcList 源集合
     * @param descType 目标集合中元素的类型
     * @return
     */
    public static <T> List<T> copyList(List srcList, Class<T> descType){
        if(srcList == null){
            return null;
        }
        List<T> descList = Lists.newArrayList();
        for(Object obj : srcList){
            T t = copy(obj, descType);
            descList.add(t);
        }
        return descList;
    }

    public static <T> Map<String, T> toMap(Object target) {
        return toMap(target,false);
    }

    /**
     * 将目标对象的所有属性转换成Map对象
     *
     * @param target 目标对象
     * @param ignoreParent 是否忽略父类的属性
     *
     * @return Map
     */
    public static <T> Map<String, T> toMap(Object target,boolean ignoreParent) {
        return toMap(target,ignoreParent,false);
    }

    /**
     * 将目标对象的所有属性转换成Map对象
     *
     * @param target 目标对象
     * @param ignoreParent 是否忽略父类的属性
     * @param ignoreEmptyValue 是否不把空值添加到Map中
     *
     * @return Map
     */
    public static <T> Map<String, T> toMap(Object target,boolean ignoreParent,boolean ignoreEmptyValue) {
        return toMap(target,ignoreParent,ignoreEmptyValue,new String[0]);
    }


    public static List<Field> getFields(Class<?> clazz, Boolean ignoreParent) {

        Field[] fields = null;

        if (ignoreParent){
            //获得某个类的所有声明的字段，即包括public、private和proteced，但是不包括父类的申明字段。
            fields = clazz.getDeclaredFields();
        }else{
            //获得某个类的所有的公共（public）的字段，包括父类中的字段。
            fields = clazz.getFields();
        }

        int len = fields.length;

        List<Field> list = Lists.newArrayList();
        for (int i = 0; i < len; i++) {
            list.add(fields[i]);
        }
        
        return list;
    }


    /**
     * 将目标对象的所有属性转换成Map对象
     *
     * @param target 目标对象
     * @param ignoreParent 是否忽略父类的属性
     * @param ignoreEmptyValue 是否不把空值添加到Map中
     * @param ignoreProperties 不需要添加到Map的属性名
     */
    public static <T> Map<String, T> toMap(Object target,boolean ignoreParent,boolean ignoreEmptyValue,String... ignoreProperties) {
        Map<String, T> map = new HashMap<String, T>();

        List<Field> fields = getFields(target.getClass(), ignoreParent);

        for (Iterator<Field> it = fields.iterator(); it.hasNext();) {
            Field field = it.next();
            T value = null;

            try {
                value = (T) field.get(target);
            } catch (Exception e) {
                e.printStackTrace();
            }

            if (ignoreEmptyValue
                    && ((value == null || value.toString().equals(""))
                    || (value instanceof Collection && ((Collection<?>) value).isEmpty())
                    || (value instanceof Map && ((Map<?,?>)value).isEmpty()))) {
                continue;
            }

            boolean flag = true;
            String key = field.getName();

            for (String ignoreProperty:ignoreProperties) {
                if (key.equals(ignoreProperty)) {
                    flag = false;
                    break;
                }
            }

            if (flag) {
                map.put(key, value);
            }
        }

        return map;
    }
}
```