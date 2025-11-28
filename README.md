# Лабораторная работа 6
/*
Задание 1.1.@Invoke.
Разработайте аннотацию @Invoke, со следующими характеристиками:
• Целью может быть только МЕТОД
• Доступна во время исполнения программы
• Не имеет свойств
Создайте класс, содержащий несколько методов, и проаннотируйте хотя бы один из них
аннотацией @Invoke.
Реализуйте обработчик (через Reflection API), который находит методы, отмеченные
аннотацией @Invoke, и вызывает их автоматически.

 Структура проекта:

 1. Аннотация @Invoke

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    
```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Invoke {
}
```

 3. Класс InvokeAnnotation с методами:
    - Несколько методов помечены @Invoke
    - Один из методов приватный (public/private значения не имеют)
```java
public class InvokeAnnotation {
    @Invoke
    public void method1(){
        System.out.println("Метод 1 вызван");
    }
    public void method2(){
        System.out.println("Метод 2 вызван");
    }
    @Invoke
    public void method3(){
        System.out.println("Метод 3 вызван");
    }
    @Invoke
    private void method4(){
        System.out.println("Метод 4 вызван");
    }
}

```
 4. Обработчик аннотации InvokeProcessor:
    - Использует Reflection API
    - Находит методы с @Invoke
    - Вызывает каждый найденный метод (method.invoke)
```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class InvokeProcessor {
    public static void processing(Object object){
        Class<?>clazz = object.getClass();
        Method[]methods = clazz.getDeclaredMethods();
        for(Method method:methods){
            if(method.isAnnotationPresent(Invoke.class)){
                try{
                    method.setAccessible(true);
                    method.invoke(object);
                } catch(IllegalAccessException | InvocationTargetException exception){
                    exception.printStackTrace();
                }
            }
        }
    }
}

```
 5. Главный класс Main:
    - Создаёт объект InvokeAnnotation
    - Передаёт его в InvokeProcessor.processing(object)
```java
public class Main {
    public static void main(String[] args) {
        InvokeAnnotation test = new InvokeAnnotation();
        InvokeProcessor.processing(test);
    }
}

```
 Результат работы программы:
 На экран выводится:

    Метод 1 вызван
    Метод 3 вызван
    Метод 4 вызван

 (Метод 2 не вызывается, т.к. не имеет аннотации @Invoke)

*/


