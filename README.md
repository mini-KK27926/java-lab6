# Лабораторная работа 6
## Задание 1.1.@Invoke.
Разработайте аннотацию @Invoke, со следующими характеристиками:
- Целью может быть только МЕТОД
- Доступна во время исполнения программы
- Не имеет свойств
Создайте класс, содержащий несколько методов, и проаннотируйте хотя бы один из них
аннотацией @Invoke.
Реализуйте обработчик (через Reflection API), который находит методы, отмеченные
аннотацией @Invoke, и вызывает их автоматически.

## Структура проекта:

### 1. Аннотация @Invoke

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
     
```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Invoke {
}
```

###  2. Класс InvokeAnnotation с методами:
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
###  3. Обработчик аннотации InvokeProcessor:
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
 ### 4. Главный класс Main:
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
### Результат работы программы:
 На экран выводится:

    Метод 1 вызван
    Метод 3 вызван
    Метод 4 вызван

 (Метод 2 не вызывается, т.к. не имеет аннотации @Invoke)

## Задание 1.2.@Default.
Разработайте аннотацию @Default, со следующими характеристиками:
- Целью может быть ТИП или ПОЛЕ
- Доступна во время исполнения программы
- Имеет обязательное свойство value типа Class
Проаннотируйте какой-либо класс данной аннотацией, указав тип по умолчанию.
Напишите обработчик, который выводит имя указанного класса по умолчанию.
## Структура проекта:

### 1. Аннотация @Default

@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)

```java
import java.lang.annotation.*;
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Default {
    Class<?> value();
}
```
###  2. Класс DefaultAnnotation с аннотацией @Default
    - Аннотация на классе: @Default(String.class)
    Поля класса:
    - someNumber — с аннотацией @Default(Integer.class)
    - anotherNumber — без аннотации
    - items — с аннотацией @Default(List.class)
```java
@Default(String.class)
public class DefaultAnnotation {
    @Default(Integer.class)
    private int someNumber;
    int anotherNumber;

    @Default(java.util.List.class)
    private java.util.List<String> items;
}

```
###  3. Обработчик аннотации DefaultProcessor:
    - Использует Reflection API
    - Выводит тип по умолчанию для класса и каждого поля, где стоит @Default
    - Игнорирует поля без аннотации
```java
public class DefaultProcessor {
    public static void printDefaultClass(Class<?> clazz) {
        Default annotation = clazz.getAnnotation(Default.class);
        if (annotation != null) {
            System.out.println("Default class: " + annotation.value().getName());
        }

        for (java.lang.reflect.Field field : clazz.getDeclaredFields()) {
            Default fieldAnnotation = field.getAnnotation(Default.class);
            if (fieldAnnotation != null) {
                System.out.println("Field " + field.getName() + " default class: " + fieldAnnotation.value().getName());
            }
        }
    }
}

```
 ### 4. Главный класс Main:
    - Вызывает обработчик DefaultProcessor для класса DefaultAnnotation
    - Выводит типы по умолчанию для класса и полей
```java
public class Main {
    public static void main(String[] args) {
        DefaultProcessor.printDefaultClass(DefaultAnnotation.class);
    }
}

```
### Результат работы программы:

 На экран выводится:

    Default class для класса DefaultAnnotation: java.lang.String
    Default class для поля someNumber: java.lang.Integer
    Default class для поля items: java.util.List

Поле anotherNumber не выводится, так как не имеет аннотации @Default
Класс DefaultAnnotation выводится с типом по умолчанию String
## Задание 1.3.@ToString.
Разработайте аннотацию @ToString, со следующими характеристиками:
- Целью может быть ТИП или ПОЛЕ
- Доступна во время исполнения программы
- Имеет необязательное свойство valuec двумя вариантами значений: YES или NO
- Значение свойства по умолчанию: YES
Проаннотируйте класс аннотацией @ToString, а одно из полей – с @ToString(Mode.NO).
Создайте метод, который формирует строковое представление объекта, учитывая только те поля,
где @ToString имеет значение YES.
## Структура проекта:

### 1. Аннотация @ToString

Применяется к классу или полю (@Target({ElementType.TYPE, ElementType.FIELD}))
Доступна во время выполнения (@Retention(RetentionPolicy.RUNTIME))
Имеет необязательное свойство value типа Mode (YES/NO), по умолчанию YES
     
```java
enum Mode {
    YES, NO
}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD})
@interface ToString {
    Mode value() default Mode.YES;
}
```

###  2.  Класс Person с полями:
    name — включено в вывод (Mode.YES)
    age — включено в вывод (Mode.YES)
    secret — исключено из вывода (@ToString(Mode.NO))
    Аннотация на классе: @ToString
```java

```
###  3. Обработчик ToStringProcessor:
    - Использует Reflection API
    - Формирует строковое представление объекта
    - Включает только поля с Mode.YES или без аннотации
```java
class ToStringProcessor {
    public static String objectToString(Object obj) {
        Class<?> clazz = obj.getClass();
        if (clazz.getAnnotation(ToString.class) == null)
            return obj.toString();

        StringBuilder sb = new StringBuilder(clazz.getSimpleName() + "{");
        boolean first = true;
        for (java.lang.reflect.Field field : clazz.getDeclaredFields()) {
            ToString ts = field.getAnnotation(ToString.class);
            Mode mode = (ts == null) ? Mode.YES : ts.value();
            if (mode == Mode.YES) {
                field.setAccessible(true);
                try {
                    if (!first) sb.append(", ");
                    sb.append(field.getName()).append("=").append(field.get(obj));
                    first = false;
                } catch (IllegalAccessException e) { /* обработка ошибки */ }
            }
        }
        sb.append("}");
        return sb.toString();
    }
}

```
 ### 4. Главный класс Main:
    - Создаёт объект Person с заданными значениями полей
    - Передаёт объект в ToStringProcessor.objectToString()
    - Выводит строковое представление объекта с учётом аннотаций @ToString на полях
```java
public class Main {
    public static void main(String[] args) {
        Person person = new Person("Елена", 22, "hidden");
        System.out.println(ToStringProcessor.objectToString(person));
    }
}
```
### Результат работы программы:
 На экран выводится:

    Person{name=Елена, age=22}
