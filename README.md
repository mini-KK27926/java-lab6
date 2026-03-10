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
@ToString
class Person {
    private String name;
    private int age;

    @ToString(Mode.NO)
    private String secret;

    public Person(String name, int age, String secret) {
        this.name = name;
        this.age = age;
        this.secret = secret;
    }
}
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
    
## Задание 1.4.@Validate.
Разработайте аннотацию @Validate, со следующими характеристиками:
- Целью может быть ТИП или АННОТАЦИЯ
- Доступна во время исполнения программы
- Имеет обязательное свойство value, типа Class[]
Проаннотируйте класс аннотацией @Validate, передав список типов для проверки.
Реализуйте обработчик, который выводит, какие классы указаны в аннотации.

### 1. Аннотация @Validate

* Применяется к классу или аннотации (`@Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})`)
* Доступна во время выполнения (`@Retention(RetentionPolicy.RUNTIME)`)
* Свойство `value` — массив классов для проверки

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})
public @interface Validate {
    Class<?>[] value();
}
```

### 2. Класс ValidateAnnotation с аннотацией @Validate

* Аннотация на классе: `@Validate({String.class, Integer.class, Double.class})`

```java
@Validate({String.class, Integer.class, Double.class})
public class ValidateAnnotation { }
```

### 3. Обработчик ValidateProcessor

* Использует Reflection API
* Считывает аннотацию `@Validate` и выводит список классов

```java
public class ValidateProcessor {
    public static void printValidatedClasses(Class<?> clazz) {
        Validate annotation = clazz.getAnnotation(Validate.class);
        if (annotation != null) {
            System.out.println("Класс " + clazz.getName() + " имеет @Validate с типами:");
            for (Class<?> type : annotation.value()) {
                System.out.println(" - " + type.getName());
            }
        }
    }
}
```

### 4. Главный класс Main

* Передаёт класс `ValidateAnnotation` в обработчик
* Выводит список классов из аннотации

```java
public class Main {
    public static void main(String[] args) {
        ValidateProcessor.printValidatedClasses(ValidateAnnotation.class);
    }
}
```

### Результат работы программы

```
Класс ValidateAnnotation имеет @Validate с типами:
 - java.lang.String
 - java.lang.Integer
 - java.lang.Double
```

---

## Задание 1.5. @Two
@Two.
Разработайте аннотацию @Two, со следующими характеристиками:
- Целью может быть ТИП
- Доступна во время исполнения программы
- Имеет два обязательных свойства: first типа String и second типа int
Проаннотируйте какой-либо класс аннотацией @Two, передав строковое и числовое значения.
Реализуйте обработчик, который считывает и выводит значения этих свойств.
### 1. Аннотация @Two

- Применяется только к классу (`@Target(ElementType.TYPE)`)
- Доступна во время выполнения (`@Retention(RetentionPolicy.RUNTIME)`)
- Два обязательных свойства: `first` (String), `second` (int)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Two {
    String first();
    int second();
}
```

### 2. Класс TwoAnnotation с аннотацией @Two

- Аннотация на классе: `@Two(first="Значение строка", second=42)`

```java
@Two(first = "Значение строка", second = 42)
public class TwoAnnotation { }
```

### 3. Обработчик TwoProcessor

* Считывает аннотацию `@Two` и выводит значения `first` и `second`

```java
public class TwoProcessor {
    public static void printTwoValues(Class<?> clazz) {
        Two annotation = clazz.getAnnotation(Two.class);
        if (annotation != null) {
            System.out.println("Значение first: " + annotation.first());
            System.out.println("Значение second: " + annotation.second());
        }
    }
}
```

### 4. Главный класс Main

* Передаёт класс `TwoAnnotation` в обработчик
* Выводит значения аннотации

```java
public class Main {
    public static void main(String[] args) {
        TwoProcessor.printTwoValues(TwoAnnotation.class);
    }
}
```

### Результат работы программы

```
Значение first: Значение строка
Значение second: 42
```

---

## Задание 1.6. @Cache
Разработайте аннотацию @Cache, со следующими характеристиками:
- Целью может быть ТИП
- Доступна во время исполнения программы
- Имеет необязательное свойство value, типа String[]
- Значение свойства по умолчанию: пустой массив
Проаннотируйте класс аннотацией @Cache, указав несколько кешируемых областей.
Создайте обработчик, который выводит список всех кешируемых областей или сообщение, что
список пуст.
### 1. Аннотация @Cache

* Применяется только к классу (`@Target(ElementType.TYPE)`)
* Доступна во время выполнения (`@Retention(RetentionPolicy.RUNTIME)`)
* Свойство `value` — массив строк (по умолчанию пустой)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Cache {
    String[] value() default {};
}
```

### 2. Класс CacheAnnotation с аннотацией @Cache

* Аннотация на классе: `@Cache({"users", "items", "products"})`

```java
@Cache({"users", "items", "products"})
public class CacheAnnotation { }
```

### 3. Обработчик CacheProcessor

* Выводит список кешируемых областей или сообщение о пустом списке

```java
public class CacheProcessor {
    public static void printCacheAreas(Class<?> clazz) {
        Cache annotation = clazz.getAnnotation(Cache.class);
        if (annotation != null) {
            String[] areas = annotation.value();
            if (areas.length == 0) {
                System.out.println("Список кешируемых областей пуст.");
            } else {
                System.out.println("Кешируемые области:");
                for (String area : areas) {
                    System.out.println("- " + area);
                }
            }
        }
    }
}
```

### 4. Главный класс Main

* Передаёт класс `CacheAnnotation` в обработчик
* Выводит кешируемые области

```java
public class Main {
    public static void main(String[] args) {
        CacheProcessor.printCacheAreas(CacheAnnotation.class);
    }
}
```

### Результат работы программы

```
Кешируемые области:
- users
- items
- products
```

