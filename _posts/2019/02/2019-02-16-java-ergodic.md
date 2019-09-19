---
layout: post
title: java遍历实体类的属性和数据类型以及属性值 
category: java
tags: [java]
---



和同学接了个外包的活，由于项目中很多地方要用到poi导出excel，而每次导出都要写很多相同的代码，因为poi的cell.setCellValue();每次设置的都是不同实体bean的属性值，导致代码里很多重复的值，我在想有没有可以自动装载bean的属性及属性值的方法。首先想到的肯定是反射，但是自己写了一下没写出来，so上网查了一下，发现了这个方法，感觉不错，就记录下来了。原文链接[https://blog.csdn.net/dongzhouzhou/article/details/8446782](https://blog.csdn.net/dongzhouzhou/article/details/8446782)

原文的代码如下
    
     /**
         * 遍历实体类的属性和数据类型以及属性值
         * @param model
         * @throws NoSuchMethodException
         * @throws IllegalAccessException
         * @throws IllegalArgumentException
         * @throws InvocationTargetException
         */
        public static void reflectTest(Object model) throws NoSuchMethodException,
                        IllegalAccessException, IllegalArgumentException,
                        InvocationTargetException {
            // 获取实体类的所有属性，返回Field数组
            Field[] field = model.getClass().getDeclaredFields();
            // 遍历所有属性
            for (int j = 0; j < field.length; j++) {
                    // 获取属性的名字
                    String name = field[j].getName();
                    // 将属性的首字符大写，方便构造get，set方法
                    name = name.substring(0, 1).toUpperCase() + name.substring(1);
                    // 获取属性的类型
                    String type = field[j].getGenericType().toString();
                    // 如果type是类类型，则前面包含"class "，后面跟类名
                    System.out.println("属性为：" + name);
                    if (type.equals("class java.lang.String")) {
                            Method m = model.getClass().getMethod("get" + name);
                            // 调用getter方法获取属性值
                            String value = (String) m.invoke(model);
                            System.out.println("数据类型为：String");
                            if (value != null) {
                                    System.out.println("属性值为：" + value);
                            } else {
                                    System.out.println("属性值为：空");
                            }
                    }
                    if (type.equals("class java.lang.Integer")) {
                            Method m = model.getClass().getMethod("get" + name);
                            Integer value = (Integer) m.invoke(model);
                            System.out.println("数据类型为：Integer");
                            if (value != null) {
                                    System.out.println("属性值为：" + value);
                            } else {
                                    System.out.println("属性值为：空");
                            }
                    }
                    if (type.equals("class java.lang.Short")) {
                            Method m = model.getClass().getMethod("get" + name);
                            Short value = (Short) m.invoke(model);
                            System.out.println("数据类型为：Short");
                            if (value != null) {
                                    System.out.println("属性值为：" + value);
                            } else {
                                    System.out.println("属性值为：空");
                            }
                    }
                    if (type.equals("class java.lang.Double")) {
                            Method m = model.getClass().getMethod("get" + name);
                            Double value = (Double) m.invoke(model);
                            System.out.println("数据类型为：Double");
                            if (value != null) {
                                    System.out.println("属性值为：" + value);
                            } else {
                                    System.out.println("属性值为：空");
                            }
                    }
                    if (type.equals("class java.lang.Boolean")) {
                            Method m = model.getClass().getMethod("get" + name);
                            Boolean value = (Boolean) m.invoke(model);
                            System.out.println("数据类型为：Boolean");
                            if (value != null) {
                                    System.out.println("属性值为：" + value);
                            } else {
                                    System.out.println("属性值为：空");
                            }
                    }
                    if (type.equals("class java.util.Date")) {
                            Method m = model.getClass().getMethod("get" + name);
                            Date value = (Date) m.invoke(model);
                            System.out.println("数据类型为：Date");
                            if (value != null) {
                                    System.out.println("属性值为：" + value);
                            } else {
                                    System.out.println("属性值为：空");
                            }
                    }
            }
        } 

由于我的实体bean里有double类型，我又不想用其封装类的方法获取值，于是我在他的基础上又加入了一段代码，

     if (type.equals("double")) {
                        Method m = model.getClass().getMethod("get" + name);
                        double value = (double) m.invoke(model);
                        System.out.println("数据类型为：double");
                        if (value >0) {
                                System.out.println("属性值为：" + value);
                        } else {
                                System.out.println("属性值为：空");
                        }
                    }
                    System.out.println("属性类型为："+type); 
这样一来，就可以知道属性类型是什么，可以很好的加判断语句了。当然原文代码还有很多值得优化的地方，我这里就没有进行优化，准备等我将它与poi导出整合之后再来优化。

 
