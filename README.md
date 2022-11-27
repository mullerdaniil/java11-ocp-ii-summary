# Содержание
- [Java Fundamentals](#chapter-1-java-fundamentals)
    - [enums](#enums)
    - [Вложенные классы](#вложенные-классы)
    - [Interface members](#interface-members)
    - [Functional programming](#functional-programming)

- [Annotations](#chapter-2-annotations)

- [Generics and collections](#chapter-3-generics-and-collections)
    - [Method reference](#method-reference)
    - [Collections Framework](#collections-framework)
    - [Сортировка](#сортировка)
    - [Generics](#generics)

- [Functional programming](#chapter-4-functional-programming)
    - [Функциональные интерфейсы](#функциональные-интерфейсы)
    - [Optional](#optional)
    - [Streams](#streams)

- [Exceptions, assertions and localization](#chapter-5-exceptions-assertions-and-localization)
    - [Исключения](#исключения)
    - [Assertions](#assertions)
    - [Date and Time](#date-and-time)
    - [Internalization and localization](#internationalization-and-localization)

- [Modular applications](#chapter-6-modular-applications)
    - [Модули](#модули)
    - [Сервисы](#сервисы)

- [Concurrency](#chapter-7-concurrency)
    - [Потоки](#потоки)
    - [Потокобезопасность](#потокобезопасность)

- [I/O](#chapter-8-io)
    - [Файлы](#файлы)
    - [I/O streams](#io-streams)
    - [Сериализация](#сериализация)
    - [Вывод данных](#вывод-данных)

- [NIO.2](#chapter-9-nio2)
    - [Path](#path)
    - [NIO.2 Files](#nio2-files)
    - [Stream API](#stream-api)

- [JDBC (empty)](#chapter-10-jdbc)
- [Security (empty)](#chapter-11-security)

# Chapter 1. Java Fundamentals

## enums

- у enum есть static метод ```values()```, instance методы ```name()``` и ```ordinal()```
```java
for (Season season : Season.values()) {
    System.out.println(season.name() + " " + season.ordinal());
}
```

- в ```switch``` в ```case``` нельзя писать enum полностью, нужно только имя
```java
Season summer = Season.SUMMER;
switch (summer) {
    case WINTER:
        System.out.println("Winter!");
        break;
    case SUMMER:
        System.out.println("Summer!");
        break;
    default:
        System.out.println("Default!");
}
```

- можно писать abstract методы и их реализацию для каждого enum
```java
enum E {
    A {
        void m() {
            System.out.println("a");
        }
    },
    B {
        void m() {
            System.out.println("b");
        }
    };
    abstract void m();
}
```

- можно писать дефолтную реализацию и override версию для отдельных enum
```java
enum E {
    A {
        void m() {
            System.out.println("a");
        }
    },
    B, C, D;
    void m() {
        System.out.println("default impl");
    }
}
```

- список значений в enum должен быть объявлен первым

## Вложенные классы

- есть четыре вида вложенных классов:
    - inner
    - static nested
    - local
    - anonymous

- inner класс объявляется там же, где и поля, методы класса. Может иметь любой access modifier. Не может иметь static поля, методы, если только это не static final поля. Имеет доступ к членам внешнего класса

- к inner классу должен быть привязать внешний класс. Можно инстанциировать inner класс и так:
```java
public class A {
    class B {}

    public static void main(String[] args) {
        A a = new A();
        a.new B();
    }
}
```

- при компиляции класса с inner классом получаются два файла: ```Outer.class``` и ```Outer$Inner.class```

- static nested классу не требуется привязка к instance внешнего класса в отличие от inner класса. В целом, похож на top-level класс, однако:
    - создается namespace (обращение через внешний класс), так как класс вложен
    - может иметь любой access modifier
    - внешний класс может обращаться к полям, методам static nested

- static nested классы можно импортировать двумя способами (обычный и static import):
```java
import package1.Outer.Nested;
```
или
```java
import static package1.Outer.Nested;
```

- local класс объявляется внутри метода (или конструктора, или initializer). Подобно local переменной, local класс выходит из scope когда заканчивается метод. Соответственно, instance'ы local класса можно создавать только внутри метода.
- особенности local классов:
    - нет access modifier
    - не могут быть static и содержать static members (за исключением static final)
    - имеют доступ ко всем members внешнего класса (если объявлены в instance method)
    - могут обращаться к переменным, если те final или effectively final
- anonymous класс - специальная форма local класса, не имеющая имени. Он объявляется и инстанциируется в одном statement через ```new```.
- c помощью anonymous класса нельзя одновременно и реализовать интерфейс, и отнаследоваться от класса (если только класс не ```Object```).

- Правила доступа вложенных классов:

| Класс | Может наследовать любой класс или реализовывать любое число интерфейсов | Имеет доступ к членам внешнего класса без ссылки на него | Имеет доступ к local переменным метода |
| --- | --- | --- | --- |
| Inner | Да | Да | - |
| Static nested | Да | Нет | - |
| Local | Да | Да (если объявлен в instance методе) | Да (если final или effectively final) |
| Anonymous | Нет (только один класс или интерфейс) | Да (если объявлен в instance методе) | Да (если final или effectively final) |

## Interface members
- есть 6 разновидностей interface members:

| Имя | Обязательные модификаторы | Неявные модификаторы | В какой версии Java появились |
| --- | --- | --- | --- |
| Constant variable | - | ```public static final``` | 1.0 |
| Abstract method | - | ```public abstract``` | 1.0 |
| Default method | ```default``` | ```public ``` | 8 |
| Static method | ```static``` | ```public``` | 8 |
| Private method | ```private``` | - | 9 |
| Private static method | ```private static``` | - | 9 |

- default метод может находиться только в интерфейсе. Он не может быть static или final. Если класс наследует два default метода с одинаковой сигнатурой, то он обязан переопределить данный метод:
```java
interface A {
    default int getValue() {
        return 5;
    }
}

interface B {
    default int getValue() {
        return 7;
    }
}

class C implements A, B {
    @Override
    public int getValue() {
        return 10;
    }
}
```

- можно вызвать версию из определенного интерфейса через ```super```:

```java
interface A {
    default int getValue() {
        return 5;
    }
}

interface B {
    default int getValue() {
        return 7;
    }
}

class C implements A, B {
    @Override
    public int getValue() {
        return A.super.getValue();
    }
}
```

- static метод в интерфейсе не может быть ```abstract``` или ```final```. Static методы не наследуются и доступны в классах-реализациях только через ссылку на сам интерфейс. Получается, что проблема множественного наследования static методов не касается.

- private метод в интерфейсе могут вызывать только ```default``` и нестатические ```private``` методы интерфейса:
```java
interface I {
    private void privateMethod() {
        
    }
    
    default void defaultMethod() {
        privateMethod();
    }
    
    private void anotherPrivateMethod() {
        privateMethod();
    }
}
```

- private static метод в интерфейсе могут вызывать другие методы в интерфейсе (разумеется, кроме abstract метода).

## Functional programming

- функциональный интерфейс - это интерфейс, имеющий один ```abstract``` метод. Lambda выражение - передаваемый блок кода, подобие анонимного класса, определяющего один ```abstract``` метод.

```java
@FunctionalInterface
interface I {
    void m();
}
```

- Данный пример показывает использование lambda выражения и анонимного класса:
```java
public class Testing {
    public static void main(String[] args) {
        use(() -> System.out.println("hi"));
        use(new I() {
            @Override
            public void m() {
                System.out.println("hi");
            }
        });
    }

    static void use(I i) {
        i.m();
    }
}

@FunctionalInterface
interface I {
    void m();
}
```

- ```public``` методы, находящиеся в классе ```Object``` являются исключениями (не учитываются) при отнесению интерфейса к функциональным, исходя из того, что все классы-реализации интерфейса наследуются от ```Object```, а значит, реализуют эти методы. Например, данный интерфейс не является функциональым:
```java
interface I {
    String toString();
}
```

Пример валидного функционального интерфейса:
```java
@FunctionalInterface
interface I {
    String toString();
    boolean equals(Object o);
    int hashCode();
    void m();
}
```

- отступление про ```equals``` и ```hashCode```. Дефолтная реализация ```equals``` - использование ```==```. Если переопределяется ```equals```, то должен переопределяться и ```hashCode()```. Если ```a.equals(b) == true```, то также должно выполняться и ```a.hashCode() == b.hashCode()```.

- лямбда-выражение может иметь много форм. Скобки после ```->``` могут опускаться (при этом опускается и ```return```), если в теле один statement. Скобки при параметрах могут опускаться если всего один параметр и его тип явно не написан. Вместо типа можно использовать ```var```. Если ```var``` используется для одного параметр, то он должен использоваться и для остальных (то же применимо и к типам). 
```java
e -> e.m()
(var e) -> e.m()
(Type e) -> e.m()
(Type e) -> { return e.m(); }

e -> {}
(var e) -> {}
(Type e) -> {}

(a, b) -> {}
(var a, var b) -> {}
(Type1 a, Type2 b) -> {}
```

- в теле лямбда-выражения можно объявлять local переменные, при этом нельзя переобъявлять переменные:
```java
    void method(int e) {
        int a = 9;
        Predicate<Integer> p = e -> {   // ERROR
            int a = 4;                  // ERROR
            return e == a;
        };
    }
```

- лямбды подобно анонимным классам могут использовать ```static```, instance, local (```final``` или effectively final) переменные:
```java
    static int staticV = 1;
    int instanceV = 2;
    String s;

    void method() {
        int localV = 3;
        final int finalV = 4;
        
        instanceV = 9;

        Predicate<Integer> p = (e) -> {
            int a = staticV;
            int b = instanceV;
            int c = localV;
            int d = finalV;
            return a + b + c + d + e > 0;
        };
        
        staticV = 19;
    }
```

# Chapter 2. Annotations

- аннотация - это форма метаданных, добавляемых в код, но не являющаяся частью программы. Аннотации похожи на интерфейс, однако могут применяться к классам, к методам, к выражениям, к другим аннотациям. 

- аннотация создается через ```@interface```:
```java
@interface MyAnnotation {}
```
- после этого аннотацию можно применять:
```java
@MyAnnotation
class C {
    @MyAnnotation class D {}
    @MyAnnotation() D d = new D();
    @MyAnnotation
    C c;
}
```
- можно указать обязательный атрибут аннотации (по умолчанию аттрибуты обязательны). При этом методы в ```@interface``` не могут принимать никаких аргументов, а также возращать ```void```.
```java
@interface MyAnnotation {
    int myAttribute();
}

@MyAnnotation(myAttribute = 14)
class C {}
```

- можно сделать атрибут необязательным, указав значение по умолчанию (которое должно быть non-null constant expression)
```java
@interface MyAnnotation {
    int myAttribute() default 18;
}

@MyAnnotation(myAttribute = 14)
class C {}

@MyAnnotation
class D {}
```

- порядок указания атрибутов не важен:
```java
@interface MyAnnotation {
    int a();
    String b() default "sample";
    double c();
}

@MyAnnotation(c = 3.5, a = 9)
class C {}
```

- в качестве типа атрибута можно использовать только:
    - примитив
    - ```String```
    - ```Class```
    - ```enum```
    - другая аннотация
    - массив любых этих типов

- можно добавлять и использовать константы в аннотациях (подобно интерфейсам переменные в аннотациях неявно ```public static final```):
```java
@interface MyAnnotation {
    int CONST = 25;
}

class C {
    void use() {
        System.out.println(MyAnnotation.CONST);
    }
}
```
- аннотации можно применять к:
    - классы, интерфейсы, enums, модули
    - переменные (```static```, поля, local)
    - методы, конструкторы
    - параметры методов, конструкторов, лямбд
    - cast выражения
    - другие аннотации

- можно использовать запись параметра (атрибута) без его названия и указывать в аннотации только значение, если выполняются следующие условия:
    - аннотация содержит атрибут ```value()``` (может быть обязательным или опциональным)
    - в аннотации не должно быть других обязательных атрибутов
    - при использовании аннотации не должны указываться значения других атрибутов
```java
@interface MyAnnotation {
    int value();
    String a() default "";
    boolean b() default false;
}

@MyAnnotation(14)
class C {}
```

- атрибут может быть массивом:
```java
@interface MyAnnotation {
    int[] array();
}

@MyAnnotation(array = 5) class C {}
@MyAnnotation(array = {6}) class D {}
@MyAnnotation(array = {5, 6}) class E {}
@MyAnnotation(array = {}) class F {}
```

- применение аннотаций можно ограничить с помощью ```@Target```, указывая различные значения в атрибуте ```value()```:
```java
@Target({ElementType.METHOD, ElementType.FIELD})
@interface MembersOnly {}

class C {
    @MembersOnly
    int f;
    
    @MembersOnly void m() {}
}
```

- опция ```ElementType.TYPE_USE``` в ```@Target``` относится ко всему, где присутствует тип, в том числе: cast операции, создание через ```new```, внутри объявлений типа. Исключение для методов: ```void```.

- с помощью аннотации ```@Retention``` (указывая различные ```value```) можно регулировать жизнь аннотаций после компиляции. Варианты: ```RetentionPolicy.SOURCE```, ```RetentionPolicy.CLASS```, ```RetentionPolicy.RUNTIME```.

- аннотация ```@Documented```, применяемая к аннотациям, включает в Javadoc информацию об аннотациях, применяемых к типам.

- маркер-аннотация ```@Inherited```, примененная к классу, заставляет наследников наследовать аннотацию. В данном примере к классу ```B``` будет также применена аннотация ```MyAnnotation```:
```java
@Inherited
@interface MyAnnotation {}

@MyAnnotation
class A {}

class B extends A {}
```
- маркер-аннотация ```@Repeatable``` позволяет применять аннотацию несколько раз. Настраивается с помощью двух аннотаций:
```java
@interface MyAnnotations {
    MyAnnotation[] value();
}

@Repeatable(MyAnnotations.class)
@interface MyAnnotation {
    int a();
    int b();
}

@MyAnnotation(a = 3, b = 7)
@MyAnnotation(a = 4, b = 9)
class A {}
```

- если ```@Target``` не указан, то аннотация применяется ко всему, кроме ```TYPE_USE``` и ```TYPE_PARAMETER``` (cast операции, создание объектов, generic объявления и т.п.). Если не указан ```@Retention```, то по умолчанию - ```RetentionPolicy.CLASS```.

- аннотация ```@Deprecated``` поддерживает 2 необязательных значения: ```String since()``` и ```boolean forRemoval()```. Пример использования ```@Deprecated``` и javadoc:
```java
class C {
    /**
     * Method to print sth.
     * @deprecated Use newM() instead.
     */
    @Deprecated(since = "1.5", forRemoval = true)
    void oldM() {
        System.out.println("old");    
    }
    
    void newM() {
        System.out.println("new");
    }
}
```

- аннотация ```@SupressWarnings``` позволяет игнорировать предупреждения компилятора. При значении ```deprecation``` игнорируются предупреждения, связанные с аннотацией ```@Deprecated```. При значении ```unchecked``` игнорируются предупреждения, связанные с raw types.

- аннотация ```@SafeVarargs``` может применяться только к непереопределяемым (```private```, ```static``` или ```final```) методам, содержащим varargs параметр.

# Chapter 3. Generics and Collections

## Method reference

- method references позволяют заменять лямбды и избежать избыточности в коде. Реализуются с помощью оператора ```::```. Имеют 4 формата:
    - ```static``` методы
    - instance методы на определенном объекте
    - instance методы на параметре, определяемом в runtime
    - конструкторы

- пример вызова ```static``` методов:
```java
class B {
    public static void m(int a, int b) {}
}

class T {
    void use() {
        BiConsumer<Integer, Integer> x = B::m;
    }
}
```

- Java сама находит подходящий метод по контексту и может выдасть ошибку, если не найдет ни одного или более одного подходящего метода.

- пример method reference на instance методе определенного объекта:
```java
class B {
    public String m(int a) {
        return "[" + a + "]";
    }
}

class T {
    String use(int p) {
        B b = new B();
        Function<Integer, String> x = b::m;
        return x.apply(p);
    }
}
```

- пример лямбды и соответствующего instance method reference на параметре:
```java
BiPredicate<String, String> l = (s, p) -> s.startsWith(p);
BiPredicate<String, String> r = String::startsWith;
```

- constructor reference осуществляется с помощью ключевого слова ```new```:
```java
class B {
    public B(int x) {}
}

class T {
    void use() {
        Function<Integer, B> l = x -> new B(x);
        Function<Integer, B> r = B::new;
    }
}
```

- примеры реального использования method reference:

| Вид | Пример |
| --- | --- |
| ```static``` метод | ```Collections::sort``` |
| instance метод на определенном объекте | ```str::startsWith``` |
| instance метод на параметре, определяемом в runtime | ```String::isEmpty``` |
| конструктор | ```ArrayList::new``` |

## Collections Framework

- оператор ```<>``` позволяет сокращать код:
```java
List<Integer> longForm = new ArrayList<Integer>();
List<Integer> shortForm = new ArrayList<>();
```

- есть 4 главных интерфейса в Java Collections Framework (при этом все из этих кроме ```Map``` наследуются от интерфейса ```Collection```):
    - ```List```
    - ```Set```
    - ```Queue```
    - ```Map```

- Java не позволяет удалять элементы из списка при использовании for each loop. Бросается ```ConcurrentModificationException```.

- ```ArrayList``` - это resizable массив. Его премущество в том, что поиск происходит за константное время. Добавление или удаление работает медленнее, чем поиск. Соответственно, ```ArrayList``` хорошо подходит в случае, если операций чтения больше, чем операций записи.

- ```LinkedList``` реализует и ```List```, и ```Queue```. Его преимущество в том, что доступ, добавление, удаление в начале и конце списка происходит за константное время. Минус в работе с произвольными индексами (работа за линейное время).

- можно создавать списки через фабричные методы:

| Метод | Описание | Можно добавлять элементы? | Можно изменять элементы? | Можно удалять элементы? |
| --- | --- | --- | --- | --- |
| ```Arrays.asList(varargs)``` | fixed size список с привязкой к массиву | Нет | Да | Нет |
| ```List.of(varargs)``` | immutable список | Нет | Нет | Нет |
| ```List.copyOf(collection)``` | immutable список с копией значений оригинала | Нет | Нет | Нет |

- если изменить ```Arrays.asList```, то изменения применятся также к оригинальному массиву, и наоборот.

- методы ```List```:
    - ```boolean add(E element)```
    - ```void add(int index, E element)```
    - ```E get(int index)```
    - ```E remove(int index)```
    - ```void replaceAll(UnaryOperator<E> op)```
    - ```E set(int index, E e)```

- обход списка с помощью итератора:
```java
    void iterate(List<Integer> list) {
        Iterator<Integer> iterator = list.iterator();;
        while (iterator.hasNext()) {
            Integer item = iterator.next();
        }
    }
```

- ```HashSet``` использует хэш-таблицу (```hashCode()```) для хранения элементов. Преимущество в том, что добавление и проверка на наличие элемента выполняется за const время, однако теряется порядок добавления элементов.

- ```TreeSet``` использует дерево, в котором элементы отсортированны, однако работает медленнее, чем ```HashSet```.

```LinkedList``` - это двусторонняя очередь, однако не так эффективна, как "чистая" очередь.

- методы ```Queue```:

| Метод | Описание | Бросает исключение? |
| --- | --- | --- |
| ```boolean add(E e)``` | Добавляет элемент в конец, бросает исключение пре превышении ограничения размеров | Да |
| ```boolean offer(E e)``` | Добавляет элемент в конец | Нет |
| ```E element()``` | Возвращает head, или бросает исключение, если очередь пустая | Да |
| ```E peek()``` | Возвращает head или ```null```, если очередь пустая | Нет |
| ```E remove()``` | Возвращает и удаляет head или бросает исключение, если очередь пустая | Да |
| ```E poll()``` | Возвращает и удаляет head или ```null```, если очередь пустая | Нет |

- вместо ```Map.of()``` можно использовать ```Map.ofEntries()```:
```java
Map<String, Integer> map = Map.ofEntries(
        Map.entry("uno", 1),
        Map.entry("dos", 2),
        Map.entry("tres", 3)
);
```
<!-- merge метод !-->
- структуры, использующие сортировку, не разрешают ```null```.

- альтернативные коллекции для многопоточности:
    - ```Vector implements List```
    - ```Hashtable implements Map```
    - ```Stack implements Queue```

## Сортировка

- порядок сортировки символов: цифры, заглавные буквы, строчные буквы.

- интерфейс ```Comparable``` имеет один метод:
```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

- пример реализации интерфейса ```Comparable``` (сравнение по полю ```String name```):
```java
class Person implements Comparable<Person> {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    @Override
    public int compareTo(Person o) {
        return this.name.compareTo(o.name);
    }
}
```

- правила реализации метода ```compareTo()```:
    - ```0```, если объекты эквивалентны
    - < ```0```, если данный объект меньше аргумента
    - \> ```0```, если данный объект больше аргумента

- ```Comparable``` находится в пакете ```java.lang```, в то время как ```Comparator``` находится в пакете ```java.util``` и имеет метод ```compare()```. Оба - функциональные интерфейсы.

- можно сортировать, передавая метод в ```Comparator.comparing()```:
```java
class C {
    static int negate(int x) {
        return -x; 
    }
    
    void sort(List<Integer> list) {
        Collections.sort(list, Comparator.comparing(C::negate));
    }
}
```

- для сортировки по нескольким полям можно использовать ```thenComparing()```:
```java
class B {
    int f1;
    int f2;

    public int getF1() {
        return f1;
    }

    public int getF2() {
        return f2;
    }
}

class T {
    void comp() {
        Comparator<B> bComparator = Comparator
                .comparingInt(B::getF1)
                .thenComparing(B::getF2);
    }
}
```

<!-- reverseOrder(), naturalOrder(), reversed() !-->

- ```Collections.sort()``` принимает только объекты, реализующие ```Comparable```, однако вместе с коллекцией можно передать ```Comparator```.

- ```Collections.binarySearch(list, index)``` возвращает индекс совпадения indexMatch, а если элемент не найден, то (-insertionPoint - 1). При этом список должен быть отсортированным.

- коллекции, которые требуют элементы, реализующие ```Comparable```, проверяют их не при компиляции, а в runtime. Однако можно передать ```Comparator``` в конструктор и снять требование:
```java
class C {
    int value;
}

class T {
    void add() {
        Set<C> set = new TreeSet<>(Comparator.comparingInt(o -> o.value));
        set.add(new C());
    }
}
```

## Generics

- generics вводятся с помощью задания параметра в угловых скобках:
```java
class C<T> {
    private final T field;

    public C(T field) {
        this.field = field;
    }

    public T getField() {
        return field;
    }
}
```

- существует конвенция для названий generics. Ее суть - использовать заглавную букву:

| Буква | Тип |
| --- | --- |
| ```E``` | Элемент |
| ```K``` | ```Map``` ключ |
| ```V``` | ```Map``` значение |
| ```N``` | Число |
| ```T``` | Тип данных |
| ```S, U, V``` | Для нескольких типов |

- на самом деле после компиляции происходит type erasure, то есть все generics становятся ```Object```. Компилятор также добавляет необходимые cast для работы с erased классами.

- интерфейс также может объявить formal type параметр:
```java
interface I<T> {
    void use(T t);
}
```

- есть 3 способа реализации generic интерфейса:
    - указание generic типа в классе:
    ```java
    class A implements I<String> {
        @Override
        public void use(String s) {}
    }
    ```

    - создание generic класса:
    ```java
    class B<U> implements I<U> {
        @Override
        public void use(U u) {}
    }
    ```

    - не использовать generics (raw type):
    ```java
    class C implements I {
        @Override
        public void use(Object o) {}
    }
    ```

- существуют ограничения на использование generics:
    - нельзя использовать ```new T()```
    - нельзя создавать массив generics
    - вызывать ```instanceof```
    - использовать примитив в качестве generic типа
    - создавать ```static``` переменную generic типа

- можно создавать generic методы. Generic тип объявляется перед возвращаемым значением:
```java
class Holder<T> {
    private final T t;

    public Holder(T t) {
        this.t = t;
    }

    public T getT() {
        return t;
    }
}

class C {
    public <T> void justTake(T t) {}
    
    public <T> Holder<T> makeHolder(T t) {
        return new Holder<>(t);
    }
}
```

- можно объявить тип явно при вызове generic метода. Пример неявного и явного объявлений типа:
```java
public Holder<String> makeStringHolder() {
    String str = "sample";
    var a = makeHolder(str);
    var b = this.makeHolder(str);
    var c = this.<String>makeHolder(str);
    return c;
}
```

- generic type параметр, объявленный в методе, не зависит от generic type параметра, объявленного в классе.

- с помощью wildcards можно ограничивать generic тип. Синтаксис заключается в использовании ```?```.

| Тип | Синтаксис |
| --- | --- |
| Неограниченный | ```?``` |
| Ограниченный сверху | ```? extends type``` |
| Ограниченный снизу | ```? super type``` |

- ```?``` означает любой тип:
```java
class C {
    void use(List<?> list) {
        System.out.println(list);
    }
    
    void pass() {
        List<String> strings = new ArrayList<>();
        List<Object> objects = new ArrayList<>();
        List<C> cs = new ArrayList<>();
        List<Integer> integers = new ArrayList<>();
        
        use(strings);
        use(objects);
        use(cs);
        use(integers);
    }
}
```

- ```? extends type``` означает ограничение типов сверху:
```java
class A {}
class B extends A {}
class C extends A {}
class D extends B {}

class E<T> {}

class Test {
    void use(E<? extends A> e) {}
    
    void test() {
        use(new E<A>());
        use(new E<B>());
        use(new E<C>());
        use(new E<D>());
        use(new E<Object>());   // COMPILE ERROR
        use(new E<String>());   // COMPILE ERROR
        use(new E<Test>());     // COMPILE ERROR
    }
}
```

- в списки с неограниченными или ограниченными сверху wildcards нельзя добавлять элементы:
```java
class A {}
class B extends A {}

class C {
    void test() {
        List<?> list1 = new ArrayList<A>();
        list1.add(new A());     // COMPILE ERROR
        
        List<? extends A> list2 = new ArrayList<>();
        list2.add(new A());     // COMPILE ERROR
        list2.add(new B());     // COMPILE ERROR
    }
}
```

- ```? super type``` означает ограничение снизу:
```java
class A {}
class B extends A {}

class C {
    void use(List<? super B> list) {}
    
    void pass() {
        List<A> as = new ArrayList<>();
        List<B> bs = new ArrayList<>();
        List<Object> objects = new ArrayList<>();
        List<String> strings = new ArrayList<>();
        
        use(as);
        use(bs);
        use(objects);
        use(strings);   // COMPILE ERROR
    }
}
```

- во время компиляции проверяется возможность добавления элемента в список. В данном случае нельзя добавить ```new A()```, так как ```list``` может оказаться ```List<B>```:
```java
class A {}
class B extends A {}
class C extends B {}

class T {
    void tryToAdd() {
        List<? super B> list = new ArrayList<A>();
        list.add(new A());      // COMPILE ERROR
        list.add(new B());
        list.add(new C());
    }
}
```

# Chapter 4. Functional programming

## Функциональные интерфейсы

- основные функциональные интерфейсы:

| Интерфейс | Тип возвращаемого значения | Метод | Количество аргументов |
| --- | --- | --- | --- |
| ```Supplier<T>``` | ```T``` | ```get()``` | 0 |
| ```Consumer<T>``` | ```void``` | ```accept(T)``` | 1 |
| ```BiConsumer<T, U>``` | ```void``` | ```accept(T, U)``` | 2 |
| ```Predicate<T>``` | ```boolean``` | ```test(T)``` | 1 |
| ```BiPredicate<T, U>``` | ```boolean``` | ```test(T, U)``` | 2 |
| ```Function<T, R>``` | ```R``` | ```apply(T)``` | 1 |
| ```BiFunction<T, U, R>``` | ```R``` | ```apply(T, U)``` | 2 |
| ```UnaryOperator<T>``` | ```T``` | ```apply(T)``` | 1 |
| ```BinaryOperator<T>``` | ```T``` | ```apply(T, T)``` | 2 |

- определения ```UnaryOperator``` и ```BinaryOperator```:
```java
public interface UnaryOperator<T> extends Function<T, T> {}
public interface BinaryOperator<T> extends BiFunction<T, T, T> {}
```

- примеры использования функциональных интерфейсов:
```java
class FIs {
    private final Map<String, Integer> strToIntMap = new HashMap<>();

    void examples() {
        Supplier<LocalDateTime> s = LocalDateTime::now;
        Consumer<Long> c = System.out::println;
        BiConsumer<String, Integer> bc = strToIntMap::put;
        Predicate<String> p = String::isEmpty;
        BiPredicate<String, String> bp = String::startsWith;
        Function<String, Integer> f = Integer::parseInt;
        BiFunction<String, Integer, Character> bf = String::charAt;
        UnaryOperator<String> uo = String::toUpperCase;
        BinaryOperator<String> bo = String::concat;
    }
}
```

- во встроенных функциональных интерфейсах существуют вспомогательные convenience методы:
    - ```andThen()``` (```Consumer```)
    - ```andThen()```, ```compose()``` (```Function```)
    - ```and()```, ```or()```, ```negate()``` (```Predicate```)

- пример использования convenience методов у ```Predicate```:
```java
class FIs {
    void c() {
        List<Integer> list = List.of(-4, -3, -2, -1, 0, 1, 2, 3, 4);
        Predicate<Integer> evenTest = x -> x % 2 == 0;
        Predicate<Integer> positiveTest = x -> x > 0;
        Predicate<Integer> bothTest = evenTest.and(positiveTest);

        for (Integer number : list) {
            if (bothTest.test(number)) {
                System.out.print(number + " ");
            }
        }
    }
}

// OUTPUT:
// 2 4 
```

## Optional

- ```Optional``` - специальный объект-контейнер, который содержит или не содержит non-null значение. Например, если при расчете среднего значения элементов количество элементов равно нулю, то среднее значение не определено. В такое случае возвращается ```Optional.empty()```. Если же значение можно посчитать, тогда возвращается ```Optional.of(result)```:
```java
class C {
    static Optional<Double> average(double... values) {
        if (values.length == 0) {
            return Optional.empty();
        }

        double sum = 0;
        for (double value : values) {
            sum += value;
        }
        double count = values.length;

        return Optional.of(sum / count);
    }
}
```

- пример проверки значения в ```Optional```:
```java
void use() {
    Optional<Double> nonEmptyBox = average(3.5, 2.7, -13.3);
    Optional<Double> emptyBox = average();

    if (nonEmptyBox.isPresent()) {
        System.out.println(nonEmptyBox.get()); // -2.3666666666666667
    }

    System.out.println(emptyBox.isEmpty()); // true

    emptyBox.get(); // NoSuchElementException
}
```

- чтобы создать empty ```Optional``` в случае ```null``` используется ```ofNullable(value)```. Две эквивалентные записи:
```java
String value = null;
Optional<String> op1 = value != null ? Optional.of(value) : Optional.empty();
Optional<String> op2 = Optional.ofNullable(value);
```

- instance методы ```Optional```:
    - ```get()```
    - ```ifPresent(Consumer c)```
    - ```isPresent()```
    - ```orElse(T other)```
    - ```orElseGet(Supplier s)```
    - ```orElseThrow()```
    - ```orElseThrow(Supplier exceptionSupplier)```

## Streams

- stream - последовательность данных. Stream pipeline состоит из частей:
    - source (сюда приходит stream)
    - промежуточные операции (превращение одного stream в другой)
    - terminal операция (получение результата, после этого stream перестает быть доступным)

- методы создания stream:

| Метод | Конечный? | Описание |
| --- | --- | --- |
| ```Stream.empty() ``` | Да | Пустой stream |
| ```Stream.of(varargs) ``` | Да | Из элементов |
| ```coll.stream() ``` | Да | Из коллекции |
| ```coll.parallelStream() ``` | Да | Параллельный stream из коллекции |
| ```Stream.generate(supplier) ``` | Нет | Элемент генерируется с помощью ```supplier``` |
| ```Stream.iterate(seed, unaryOperator) ``` | Нет | Первый элемент создается с помощью ```seed```, а последующие с помощью ```unaryOperator``` |
| ```Stream.iterate(seed, predicate, unaryOperator) ``` | Да/Нет | То же самое, только остановка, если ```Predicate``` - ```false```|

- Reduction - специальный тип терминальной операции, при которой все содержимое stream преобразуется в один примитив или объект.

- терминальные операции:
    - ```count()```
    - ```min()```, ```max()```
    - ```findAny()```, ```findFirst()```
    - ```allMatch()```, ```anyMatch()```, ```noneMatch()```
    - ```forEach()```, ```reduce()```, ```collect()```

- сигнатуры метода ```reduce()```:
    - ```T reduce(T identity, BinaryOperator<T> accumulator)```
    - ```Optional<T> reduce(BinaryOperator<T> accumulator)```
    - ```<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)```

- пример использования ```reduce()```:
```java
Stream<Integer> s = Stream.of(2, 3, 5);
int p = s.reduce(1, (a, b) -> a * b);
System.out.println(p);  // 30
```

- сигнатуры метода ```collect()```:
    - ```<R> R collect(Supplier<R> supplier,BiConsumer<R, ? super T> accumulator,BiConsumer<R, R> combiner)```
    - ```<R,A> R collect(Collector<? super T, A,R> collector)```

- примеры использования ```collect()```:
```java
Stream<Integer> s = Stream.of(2, 3, 5);
StringBuilder p
        = s.collect(StringBuilder::new,
        (sb, number) -> sb.append(number + "/"),
        StringBuilder::append
        );
System.out.println(p);
```

```java
Stream<Integer> s = Stream.of(2, 3, 5);
Set<Integer> set = s.collect(Collectors.toSet());
System.out.println(set);    // [2, 3, 5]
```

- основные intermediate операции:
    - ```filter()```
    - ```distinct()```
    - ```limit()```
    - ```skip()```
    - ```map()```
    - ```flatMap()```
    - ```sorted()```
    - ```peek()```

- пример использования ```flatMap()```:
```java
List<Integer> l1 = List.of(2, 7, 11);
List<Integer> l2 = List.of(4, 8);
Stream<List<Integer>> s = Stream.of(l1, l2);
s.flatMap(Collection::stream)
    .forEach(e -> System.out.print(e + " "));
```

- можно использовать притимивные streams:
    - ```IntStream```
    - ```LongStream```
    - ```DoubleStream```

- методы primitive streams:
    - ```average()``` (```OptionalDouble```)
    - ```boxed()```
    - ```max()``` (Optional)
    - ```min()``` (Optional)
    - ```range()``` (```IntSteam```, ```LongStream```)
    - ```rangeClosed()``` (```IntSteam```, ```LongStream```)
    - ```sum()```
    - ```summaryStatistics()```

- с помощью метода ```map()``` можно получать stream такого же типа, а с ```mapToObj()```, ```mapToDouble()```, ```mapToInt()```, ```mapToLong()``` получать streams другого типа.

- интерфейс ```Supplier``` не позволяет использовать checked исключения. Как решение, можно превратить checked исключение в unchecked.

<!-- collectors !-->

# Chapter 5. Exceptions, Assertions, and Localization

## Исключения

- в Java все исключения наследуются от ```Throwable```. ```Exception``` и ```Error``` - классы-наследники ```Throwable```. ```RuntimeException``` наследуется от ```Exception```.

- Checked исключения должны быть объявлены или обработаны. ```Exception```, все исключения, которые наследуют ```Exception```, но не ```RuntimeException```, являются checked. Также checked исключениями являются ```Throwable```, его наследники, но не ```RunTimeException``` и не ```Error```.

- в try-with-resources можно передавать только типы, реализующие интерфейс ```AutoCloseable``` с методом ```void close() throws Exception```. Данный метод вызывается в конце ```try``` блока (в реальности JVM вызывает ```close()``` в неявном ```finally``` блоке). Ресурсы закрываются в порядке, обратном порядку их открытия. После закрытия ресурсы больше не видны в ```catch``` и ```finally```.

- можно использовать ```final``` или effectively final ресурсы в try-with-resources:
```java
class T {
    void test() {
        final var r1 = new Res();
        Res r2 = new Res();
        try (r1; r2) {}
    }
}

class Res implements AutoCloseable {
    @Override
    public void close() {}
}
```

<!-- supressed exceptions !-->

## Assertions
- assertions позволяют находить ошибки в коде, ожидая, что что-то будет ```true```. Синтаксис:
```java
assert test_value;
assert test_value: message;
```

- в случае ```false``` выбрасывается ```AssertionError```. Если assertions выключены, то assertions игнорируются. Если включены и ```true```, то ничего не происходит, иначе - выбрасывается ошибка.

- assertions включаются с помощью флагов при запуске:
```
java -enableassertions Program
```
или
```
java -ea Program
```

- можно указать определенные классы или пакеты, в которых включить assertions:
```
java -ea:com.pack... com.pack.Program
```

- выключение происходит с помощью флага ```-da```:
```
java -ea:com.pack... -da:com.pack.Special com.pack.Program
```

## Date and Time

- типы даты и времени:

| Класс | Описание |
| --- | --- |
| ```java.time.LocalDate``` | Дата с днем, месяцем, годом |
| ```java.time.LocalTime``` | Время дня |
| ```java.time.LocalDateTime``` | День и время без временной зоны |
| ```java.time.ZonedDateTime``` | День и время с определенной временной зоной |

- каждый тип содержит ```static``` метод ```now()```. Результат зависит от времени и места выполнения.

- можно создавать дату и время с помощью метода ```of()```:
```java
LocalDate date = LocalDate.of(2022, Month.NOVEMBER, 13);
LocalTime time = LocalTime.of(23, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time);
```

- примеры методов ```LocalDate```:
```java
LocalDate date = LocalDate.of(2022, 11, 13);
System.out.println(date.getDayOfWeek());    // SUNDAY
System.out.println(date.getMonth());        // NOVEMBER
System.out.println(date.getDayOfYear());    // 317
```

- с помощью ```DateTimeFormatter``` можно форматировать объекты даты и времени:
```java
LocalDate date = LocalDate.of(2022, 11, 13);
String str = date.format(DateTimeFormatter.ISO_DATE);
System.out.println(str);    // 2022-11-13
```

- можно использовать собственный формат:
```java
var t = LocalDateTime.of(2022, 11, 13, 18, 3);
var f = DateTimeFormatter.ofPattern("dd, MMMM, yyyy (hh:mm)");
System.out.println(t.format(f));    // 13, ноября, 2022 (06:03)
```

- также можно использовать ```SimpleDateFormat```:
```java
var d = new Date();
var f = new SimpleDateFormat("dd, MMMM, yyyy (hh:mm)");
System.out.println(f.format(d));    // 13, ноября, 2022 (06:07)
```

- date/time символы:

| Символ | Значение | Примеры |
| --- | --- | --- |
| ```y``` | Год | ```22```, ```2022``` |
| ```M``` | Месяц | ```1```, ```01```, ```Jan```, ```January``` |
| ```d``` | День | ```3```, ```03``` |
| ```h``` | Час | ```8```, ```08``` |
| ```m``` | Минута | ```15``` |
| ```s``` | Секунда | ```42``` |
| ```a``` | am/pm | ```AM, PM``` |
| ```z``` | Название time zone | ```Eastern Standard Time, EST``` |
| ```Z``` | Сдвиг time zone | ```-0400``` |

- при использовании несовместимых символов можно получить ```DateTimeException```:
```java
var d = LocalDateTime.of(2005, 12, 16, 18, 45);
var f = DateTimeFormatter.ofPattern("hh:mm z");
System.out.println(f.format(d));

// Exception in thread "main" java.time.DateTimeException: Unable to extract ZoneId from temporal 2005-12-16T18:45
```

- метод ```format()``` можно вызывать как у DateTime, так и у Formatter объектов:
```java
var d = LocalDateTime.of(2005, 12, 16, 18, 45);
var f = DateTimeFormatter.ofPattern("hh : mm");
System.out.println(f.format(d));    // 06 : 45
System.out.println(d.format(f));    // 06 : 45
```

- для вставки другого текста в Pattern нужно использовать ```'```:
```java
var d = LocalDateTime.of(2005, 12, 16, 18, 45);
var f = DateTimeFormatter.ofPattern("'Date: 'dd.MM.yy, 'Time':hh:mm:ss, 'Escape symbol: '''");
System.out.println(f.format(d));    // Date: 16.12.05, Time:06:45:00, Escape symbol: '
```

## Internationalization and Localization

- интернационализация - процесс дизайна программы так, чтобы она могла адаптироваться. Локализация - поддержка множества мест или географических регионов. Она включает в себя перевод строк на разные языки, вывод дат и чисел в правильном для региона формате.

- locale - определенный географический, политический или культурный регион. Класс ```Locale``` находится в пакете ```java.util```. Можно получить текущий locale пользователя:
```java
Locale locale = locale.getDefault();
System.out.println(locale); // ru_RU
```

- в выводе сначала идет lowercase код языка (всегда обязателен). Затем идет ```_```, за которым следует uppercase код страны (опционален). Два формата:
```
ru
en_US
```

- есть встроенный константы для locales. Также locales можно создавать через конструктор:
```java
System.out.println(Locale.GERMAN);          // de
System.out.println(Locale.GERMANY);         // de_DE
System.out.println(new Locale("de"));       // de
System.out.println(new Locale("de", "DE")); // de_DE
```

- locale можно создать через builder:
```java
Locale l = new Locale.Builder()
        .setLanguage("ru")
        .setRegion("RU")
        .build();
```

- можно установить locale по умолчанию. Тогда эти изменения будут применяться только к данной программе:
```java
Locale locale = new Locale("fr");
Locale.setDefault(locale);
```

- для локализации чисел нужно использовать ```NumberFormat```. Пример использования:
```java
int num = 23_500_000;
var f1 = NumberFormat.getNumberInstance(Locale.US);
var f2 = NumberFormat.getNumberInstance(Locale.FRANCE);
System.out.println(f1.format(num)); // 23,500,00
System.out.println(f2.format(num)); // 23 500 00
```

- существует несколько фабричных методов для получения ```NumberFormat```. Во все можно передать либо объект ```Locale```, либо не передавать аргументы:
    - ```getInstance()```
    - ```getNumberInstance()```
    - ```getCurrencyInstance()```
    - ```getPercentInstance()```
    - ```getIntegerInstance()```

- с помощью ```parse()``` можно парсить строку в число:
```java
String s = "40.45";
var en = NumberFormat.getInstance(Locale.US);
System.out.println(en.parse(s)); // 40.45
var fr = NumberFormat.getInstance(Locale.FRANCE);
System.out.println(fr.parse(s)); // 40
```

- можно использовать собственный formatter, где используются два основных символа: ```#``` - пропустить позицию, если на ней нет цифры, ```0``` - поставить 0, на позицию, если на ней нет цифры. Примеры:
```java
double d = 1234567.467;
NumberFormat f1 = new DecimalFormat("###,###,###.0");
System.out.println(f1.format(d)); // 1,234,567.5

NumberFormat f2 = new DecimalFormat("000,000,000.00000");
System.out.println(f2.format(d)); // 001,234,567.46700

NumberFormat f3 = new DecimalFormat("$#,###,###.##");
System.out.println(f3.format(d)); // $1,234,567.47
```

- пример использования ```DateTimeFormatter```:
```java
var dt = LocalDateTime.now();
var f = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT);
String str = f.withLocale(Locale.getDefault())
               .format(dt);
System.out.println(str);    // 14.11.2022, 8:31
```

## Properties

- resource bundle содержит специальные local объекты. Resource bundle в основном хранится в properties файлах. Properties файл - файл в специальном формате с парами ключ/значение.

- resource bundle можно получить двумя способами:
```java
ResourceBundle.getBundle("name");
ResourceBundle.getBundle("name", locale);
```

- при поиске resource bundle Java пытается найти самое точное значение. Если искомый locale - это ```new Locale("fr", "FR")```, а по умолчанию - ```Locale.US```, то порядок поиск такой (сначала искомый locale, затем ```Local.getDefault()```):
    - ```Res_fr_FR.properties```
    - ```Res_fr.properties```
    - ```Res_en_US.properties```
    - ```Res_en.properties```
    - ```Res.properties```
    - ```MissingResourceException``` (ничего не было найдено)

- найденный resource bundle будет использоваться как основной. Однако при поиске значений в основном resource bundle может не оказаться нужного значения. Тогда Java будет искать первое нужное значение только в одной ветви иерархии. В данном примере locale по умолчанию - ```Locale.US```:
    - если выбран ```ResC_fr_FR.properties```, как основной, то порядок поиска будет: ```ResC_fr_FR.properties```, ```ResC_fr.properties```, ```ResC.properties```.
    - если выбран ```ResC_fr.properties```, как основной, то порядок поиска будет: ```ResC_fr.properties```, ```ResC.properties```.

- с помощью ```MessageFormat``` можно подставлять строки:
```java
var msg = "The name is {0}.";
System.out.println(MessageFormat.format(msg, "Jack")); // The name is Jack.
```

- можно использоваться ```Properties``` для хранения пар ключ/значения. Пример использования:
```java
var props = new Properties();
props.setProperty("name", "Jack");
props.setProperty("surname", "Johnson");
System.out.println(props.getProperty("name"));  // Jack
System.out.println(props.getProperty("non-existing"));  // null
System.out.println(props.getProperty("non-existing2", "defaultValue")); // defaultValue
```

- только метод ```getProperty()``` позволяет передавать значение по умолчанию. ```Properties``` наследует ```Map<Object, Object>```, однако не рекомендуется вызывать ```get()```/```put()``` у ```Properties```.

# Chapter 6. Modular Applications

## Модули

- существует несколько типов модулей:
    - named
    - automatic
    - unnamed

- named модуль - модуль, содержащий ```module-info``` файл. Этот файл находится в корне JAR. Такие модули находятся в module path, но не в classpath.

- automatic модули также находятся в module path, но не содержат ```module-info``` файл. Несмотря на то, что у automatic модулей не указано имя, Java все равно определяет его по имени JAR по следующему алгоритму:
    - если ```MANIFEST.MF``` определяет ```Automatic-Module-Name```, то используется это имя. Иначе, перейти к следующему шагу:
    - удалить расширение ```.jar```
    - удалить версию (например, ```-1.0.0``` или ```-1.0-SNAPSHOT```)
    - заменить символы, не являющиеся буквами, на ```.```
    - заменить последовательность ```.``` на единственный символ ```.```
    - удалить ```.```, если это первый или последний символ строки

- unnamed модули находятся в classpath. Это обычные файлы JAR. В отличие от automatic, unnamed модули находятся в classpath, а не в module path. Они могут содержать ```module-info```, но он игнорируется. Модули в module path не могут читать unnamed модули.

- встроенный модуль ```java.base``` содержит основные используемые пакеты. Данный модуль не обязательно объявлять в ```requires```, так как он подключается автоматически.

- команда ```jdeps``` позволяет получать информацию о зависимостях. Пример вывода:
```
jdeps zoo.dino.jar
zoo.dino.jar -> java.base
zoo.dino.jar -> jdk.unsupported
 zoo.dinos -> java.lang java.base
 zoo.dinos -> java.time java.base
 zoo.dinos -> java.util java.base
 zoo.dinos -> sun.misc JDK internal API (jdk.unsupported)
```

- также ```jpeps``` можно запустить в режиме summary:
```
jdeps -s zoo.dino.jar
zoo.dino.jar -> java.base
zoo.dino.jar -> jdk.unsupported
```

- модульная система не позволяет использовать циклические зависимости между модулями.

## Сервисы

- сервис состоит из интерфейса, любых классов, которые использует интерфейс, и способ поиска реализаций интерфейса. Реализации не являются частью сервиса.

- service provider interface - интерфейс, определяющий поведение сервиса. Например, модуль может содержать один интерфейс с несколькими методами, который модуль экспортирует с помощью ```exports```.

- service locator может находить любой класс, реализующий service provider interface:
```java
ServiceLoader<MyServProvI> loader = ServiceLoader.load(MyServProvI.class);
```

- consumer (клиент) - модуль, который использует сервис. Consumer получает сервис с помощью service locator, а затем вызывает методы, которые существуют у service provider interface.

- service provider - реализация service provider interface. Для предоставления интерфейса в модуле, содержащем реализацию, нужно использовать ```provides```:
```java
provides com.pack.api.MyServProvI with com.pack2.MyServProvIImpl;
```

- характеристики артифактов для сервиса:

| Артифакт | Часть сервиса? | Необходимые директивы в ```module-info.java``` |
| --- | --- | --- |
| Service provider interface | Да | ```exports``` |
| Service provider | Нет | ```requires```, ```provides``` |
| Service locator | Да | ```exports```, ```requires```, ```uses``` |
| Consumer | Нет | ```requires``` |

# Chapter 7. Concurrency

## Потоки

- поток (thread) - наименьшая единица выполнения, которая может быть запланирована операционной системой. Процесс - группа связанных потоков, которые выполняются в одной shared среде. Single-threaded процесс - процесс, имеющий ровно один поток. Multithreaded процесс может содержать один или более потоков.

- shared среда означает, что потоки в одном процессе делят одну память и могут напрямую общаться друг с другом.

- task - единица работы, выполняемой потоком. Поток может выполнить несколько независимых tasks, но только одну task одновременно.

- системный поток создается JVM и выполняется на фоне приложения. Обычно выполнения системных потоков не видно разработчику приложения. Когда поток сталкивается с проблемой, то генерирует ```Error```.

- операционные системы используют thread scheduler для того, чтобы определять, какие потоки должны исполняться сейчас. Context switch (переключение) - процесс сохранения текущего состояния потока и последующего восстановления состояния вместе с возобновлением его выполнения.

- потоки иметь приоритет (thread priority) - целочисленное значение, определяющее приоритет выполнения потока, определяемый thread scheduler.

- интерфейс ```Runnable``` используется для определения задачи, которую поток будет выполнять:
```java
@FunctionalInterface
public interface Runnable {
 void run();
}
```

- в силу того, что ```Runnable``` - функциональный интерфейс, реализация может быть как в виде лямбды, так и в виде класса. Второй вариант используется для передачи параметров через конструктор.

- определить task, которую поток будет выполнять, можно способами:
    - передать ```Runnable``` в конструктор ```Thread```
    - создать класс, расширяющий ```Thread``` и переопределить ```run()```

- поток запускается с помощью метода ```start()``` (но не ```run()```). При этом порядок выполнения нескольких потоков нельзя определить наверняка.

- с помощью Concurrency API можно создать объект ```ExecutorService```, позволяющий выполнять tasks:
```java
Runnable task = () -> System.out.println("Do the task");
ExecutorService service = null;
try {
    service = Executors.newSingleThreadExecutor();
    service.execute(task);
    service.execute(task);
} finally {
    if (service != null) {
        service.shutdown();
    }
}
```

- метод ```shutdown()``` отклоняет новые задачи для выполнения, при этом оставшие задачи продолжают выполняться. На этом этапе ```isShutdown()``` возвращает ```true```, а ```isTerminated()``` - ```false```. При отправке новой задачи будет выброшено ```RejectedExecutionException```. Как только все задачи выполнятся, ```isShutdown()``` и ```isTerminated()``` будут возвращать ```true```.

- метод ```shutdownNow()``` пытается остановить все выполняющиеся и еще не начатые задачи, а также возвращается ```List<Runnable>``` - список еще не начатых задач.

- существует несколько методов запуска задач:

| Метод | Описание |
| --- | --- |
| ```void execute(Runnable command) ``` | ```Runnable``` выполняется к какому-либо моменту времени |
| ```Future<?> submit(Runnable task) ``` | ```Runnable``` выполняется к какому-либо моменту времени, возвращается ```Future```, содержащий результат выполнения |
| ```<T> Future<T> submit(Callable<T> task)``` | ```Callable``` выполняется к какому-либо моменту времени, возвращается ```Future```, содержащий результат выполнения |
| ```<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)throws InterruptedException``` | Выполняются все задачи, ожидается завершение выполнения всех задач. Возвращается список ```Future``` в том же порядке |
| ```<T> T invokeAny(Collection<? extendsCallable<T>> tasks) throwsInterruptedException,ExecutionException``` | Выполняются все задачи, ожидается завершение выполнения хотя бы одной задачи. Возвращается ```Future``` для завершенной задачи и отменяются все незавершенные задачи |

- в отличие от ```execute()```, ```submit()``` возвращает результат выполнения (```Future```), а также позволяет использовать ```Callable```.

- методы у ```Future```:
    - ```boolean isDone()```
    - ```boolean isCancelled() ```
    - ```boolean cancel(boolean mayInterruptIfRunning)```
    - ```V get()```
    - ```V get(long timeout, TimeUnit unit)```

- значения ```TimeUnit```:
    - ```TimeUnit.NANOSECONDS```
    - ```TimeUnit.MICROSECONDS```
    - ```TimeUnit.MILLISECONDS```
    - ```TimeUnit.SECONDS```
    - ```TimeUnit.MINUTES```
    - ```TimeUnit.HOURS```
    - ```TimeUnit.DAYS```

- интерфейс ```Callable``` похож на ```Runnable```, однако его метод возвращает значение:
```java
@FunctionalInterface public interface Callable<V> {
    V call() throws Exception;
}
```

- метод ```awaitTermination()``` ждет определенное время до завершения всех задач. Возвращает ```true```, если все задачи выполнились. Если ```awaitTermination()``` вызывается до ```shutdown()```, то поток будет ждать полное время, переданное в ```shutdown()```.

- с помощью ```ScheduledExecutorService``` можно планировать выполнение задач:
```java
ScheduledExecutorService service
 = Executors.newSingleThreadScheduledExecutor();
```

- методы ```ScheduledExecutorService``` (все возвращают ```ScheduledFuture```):
    - ```schedule(Callable<V> callable, long delay, TimeUnit unit)```
    - ```schedule(Runnable command, long delay, TimeUnit unit)```
    - ```scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)```
    - ```scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)```

- thread-pool - это группа pre-instantiated переиспользуемых потоков. Фабричные методы ```Executors``` для создания потоков:
 - ```ExecutorService newSingleThreadExecutor()```
 - ```ScheduledExecutorService newSingleThreadScheduledExecutor()```
 - ```ExecutorService newCachedThreadPool()```
 - ```ExecutorService newFixedThreadPool(int)```
 - ```ScheduledExecutorService newScheduledThreadPool(int)```

 - по сути, ```newFixedThreadPool(1)``` эквивалентен ```newSingleThreadExecutor()```. ```newCachedThreadPool()``` создает пул неограниченного размера, при этом создавая новый поток при необходимости.

## Потокобезопасность

 - потокобезопасность - свойство объекта, которое гарантирует безопасное выполнение несколькими потоками одновременно.

 - оператор ``++`` (pre-increment) не является потокобезопасным., потому что эта операция не является атомарной, а делится на две задачи - прочитать и записать, что могут прервать другие потоки. Атомарность - свойство операции быть неделимой единицей выполнения, не быть прерванной другими потоками.

 - существуют атомарные классы, эквивалентные обычным примитивам:
    - ```AtomicBoolean```
    - ```AtomicInteger```
    - ```AtomicLong```

- атомарные методы:
    - ```get()```
    - ```set()```
    - ```getAndSet()```
    - ```incremenetAndGet()```
    - ```getAndIncrement()```
    - ```decrementAndGet()```
    - ```getAndDecrement()```

- пример использования ```AtomicInteger```:
```java
private AtomicInteger count = new AtomicInteger(0);
private void incAndPrint() {
    System.out.print((count.incrementAndGet()) + " ");
}
```

- в ```synchronized``` блок может войти одновременно только один поток. Как только поток завершает выполнения блока, другие потоки могут войти в этот блок.

- пример с использованием ```synchronized``` блока. В данном случае числа выводятся по порядку:
```java
class Manager {
    private AtomicInteger count = new AtomicInteger(0);
    private void incAndPrint() {
        synchronized (this) {
            System.out.print((count.incrementAndGet()) + " ");
        }
    }

    public static void main(String[] args) {
        ExecutorService service = null;
        try {
            Manager manager = new Manager();
            service = Executors.newFixedThreadPool(20);
            for (int i = 0; i < 10; i++) {
                service.submit(manager::incAndPrint);
            }
        } finally {
            if (service != null) {
                service.shutdown();
            }
        }
    }
}

// 1 2 3 4 5 6 7 8 9 10
```

- можно использовать ```synchronized``` на методе. Две эквивалентные записи:
```java
private void incrementAndReport() {
    synchronized(this) {
        System.out.print((++sheepCount)+" ");
    }
}

private synchronized void incrementAndReport() {
    System.out.print((++sheepCount)+" ");
}
```

- также ```synchronized``` можно использовать на ```static``` методах. Тогда монитором будет выступать объект ```MyClass.class```.

- вместо ```synchronized``` можно использовать интерфейс ```Lock```, имеющий методы ```lock()``` и ```unlock()```. Пример использования:
```java
private static Lock lock = new ReentrantLock();
private AtomicInteger count = new AtomicInteger(0);
private void incAndPrint() {
    try {
        lock.lock();
        System.out.print((count.incrementAndGet()) + " ");
    } finally {
        lock.unlock();
    }
}
```

- хорошая практика, использовать ```try```/```finally``` конструкцию с ```Lock```, чтобы locks правильно освобождались.

- в случае попытки отпустить неимеющийся у потока lock бросается исключения:
```java
Lock lock = new ReentrantLock();
lock.unlock(); // IllegalMonitorStateException
```

- методы ```Lock```:

| Метод | Описание |
| --- | --- |
| ```void lock()``` | Запрашивает lock и блокируется, пока lock не будет получен |
| ```void unlock()``` | Освождает lock |
| ```boolean tryLock()``` | Запрашивает lock и сразу возвращает. Возвращает, был ли получен lock |
| ```boolean tryLock(long,TimeUnit)``` | Запрашивает lock и блокируется на указанное время, пока lock не будет получен. Возвращает, был ли получен lock |

- пример использования ```tryLock```:
```java
Lock lock = new ReentrantLock();
new Thread(() -> printMessage(lock)).start();
if (lock.tryLock()) {
    try {
        System.out.println("Lock obtained, entering protected code");
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Unable to acquire lock, doing something else");
}
```

- ```CyclicBarrier``` позволяет выполнять задачи по порядку. Он ждет выполнения этапа необходимым количеством потоков, и затем выполнение продолжается. Пример:
```java
class Manager {
    private final int executorsCount;
    private CyclicBarrier barrier1;
    private CyclicBarrier barrier2;

    public Manager(int executorsCount) {
        this.executorsCount = executorsCount;
        this.barrier1 = new CyclicBarrier(executorsCount);
        this.barrier2 = new CyclicBarrier(executorsCount, () -> System.out.println("task #2 finished"));
    }

    private void doTask1() {
        System.out.println("Doing task #1");
    }

    private void doTask2() {
        System.out.println("Doing task #2");
    }

    private void doTask3() {
        System.out.println("Doing task #3");
    }

    public void doTasks() {
        try {
            doTask1();
            barrier1.await();
            doTask2();
            barrier2.await();
            doTask3();
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        ExecutorService service = null;
        try {
            int executorsCount = 4;
            
            service = Executors.newFixedThreadPool(executorsCount);
            Manager manager = new Manager(executorsCount);
            for (int i = 0; i < executorsCount; i++) {
                service.submit(manager::doTasks);
            }
        } finally {
            if (service != null) {
                service.shutdown();
            }
        }
    }
}

/*
Doing task #1
Doing task #1
Doing task #1
Doing task #1
Doing task #2
Doing task #2
Doing task #2
Doing task #2
task #2 finished
Doing task #3
Doing task #3
Doing task #3
Doing task #3
*/
```

- использование nonconcurrent коллекции может повлечь ```ConcurrentModificationException``` даже с одним потоком:
```java
var foodData = new HashMap<String, Integer>();
foodData.put("penguin", 1);
foodData.put("flamingo", 2);
for (String key: foodData.keySet())
    foodData.remove(key);
```

- существуют concurrent аналоги коллекций. При их использовании предыдущее исключение не выбрасывается:
```java
var foodData = new ConcurrentHashMap<String, Integer>();
foodData.put("penguin", 1);
foodData.put("flamingo", 2);
for(String key: foodData.keySet())
    foodData.remove(key);
```

<!-- skip list / copy on write collections !-->
<!-- blocking queue !-->
<!-- synchronized collection !-->

- можно выделить 3 проблемы своевременности выполнения программы:
    - deadlock (два или более потока блокируются навсегда, ожидая друг друга)
    - starvation (один поток не может получить доступ к shared ресурсу или lock, постоянно занятый другими потоками)
    - livelock (два или более потока блокируются, хотя все еще активны и пытаются завершить выполнение)

- race condition - ситуация, при которой две задачи, которые должны выполняться последовательно, выполняются одновременно.

- parallel stream - stream, способный обрабатывать результаты одновременно, используя несколько потоков. Можно создать parallel stream из существующего stream или вызвать метод у коллекции:
```java
Stream<Integer> s1 = List.of(1,2).stream();
Stream<Integer> s2 = s1.parallel();

Stream<Integer> s3 = List.of(1,2).parallelStream();
```

- ```Stream``` имеет метод ```isParallel()```, позволяющий проверить, поддерживает ли объект параллельную обработку. 

- parallel decomposition - процесс разбиения задачи на подзадачи, которые могут быть выполнены параллельно, и затем, сборки результатов.

- есть альтернативная версия ```forEach()``` - ```forEachOrdered()```, при которой parallel stream обрабатывает результаты по порядку.

<!-- parallel reductions !-->
<!-- parallel reductions on collector !-->
<!-- stateless lambdas !-->

# Chapter 8. I/O

## Файлы

- ```java.io.File``` используется, чтобы прочитать информацию о существующих файлах и директориях, вывести содержание директории, и создавать/удалять файлы и директории. Instance класса ```File``` представляет путь до определенного файла или директории в файловой системе. С помощью ```File``` нельзя читать или писать данные в файле, однако можно передавать его как ссылку в потоковые классы.

- можно получить локальный разделительный символ двумя способами:
```java
System.out.println(System.getProperty("file.separator"));
System.out.println(java.io.File.separator);
```

- методы ```File```:
    - ```boolean delete()```
    - ```boolean exists()```
    - ```String getAbsolutePath()```
    - ```String getName()```
    - ```String getParent()```
    - ```boolean isDirectory()```
    - ```boolean isFile()```
    - ```long lastModified()```
    - ```long length()```
    - ```File[] listFiles()```
    - ```boolean mkdir()```
    - ```boolean mkdirs()```
    - ```boolean renameTo(File dest)```

## I/O Streams

- содержимое файла может быть прочитано или записано с помощью потока (stream). При использовании потоков неизестно, где находятся начало или конец файла. Известна лишь текущая позиция в потоке. Каждый поток разделяет данные на блоки определенным образом.

- существует два множества классов потоков:
    - Byte потоки: чтение/запись двоичных данных, имена классов заканчиваются на ```InputStream```, ```OutputStream```
    - Character потоки: чтение/запись текстовых данных, именна классов заканчиваются на ```Reader```, ```Writer```

- существует 4 абстрактных класса, являющихся родительскими всех классов потоков:
    - ```InputStream```
    - ```OutputStream```
    - ```Reader```
    - ```Writer```

- классы потоков в ```java.io```:
    - ```FileInputStream```
    - ```FileOutputStream```
    - ```FileReader```
    - ```FileWriter```
    - ```BufferedInputStream```
    - ```BufferedOutptuStream```
    - ```BufferedReader```
    - ```BufferedWriter```
    - ```ObjectInputStream```
    - ```ObjectOutputStream```
    - ```PrintStream```
    - ```PrintWriter```

- самые важные методы ```read()``` и ```write()```, позволяющие прочитать/записать байт. ```int``` используется из-за специальных значений, таких как ```-1``` (конец потока):
```java
// InputStream and Reader
public int read() throws IOException

// OutputStream and Writer
public void write(int b) throws IOException
```

- можно читать/писать массив байт:
```java
// InputStream
public int read(byte[] b) throws IOException
public int read(byte[] b, int offset, int length) throws IOException

// OutputStream
public void write(byte[] b) throws IOException
public void write(byte[] b, int offset, int length) throws IOException

// Reader
public int read(char[] c) throws IOException
public int read(char[] c, int offset, int length) throws IOException

// Writer
public void write(char[] c) throws IOException
public void write(char[] c, int offset, int length) throws IOException
```

- при использовании wrapped потоков, необязательно закрывать каждый поток, так как можно вызвать ```close()``` на самом внешнем объекте. При этом закроются и все внутренние:
```java
try (var fis = new FileOutputStream("zoo-banner.txt"); // Unnecessary
        var bis = new BufferedOutputStream(fis);
        var ois = new ObjectOutputStream(bis)) {
    ois.writeObject("Hello");
}

// Instead, do this
try (var ois = new ObjectOutputStream(
        new BufferedOutputStream(
                new FileOutputStream("zoo-banner.txt")))) {
    ois.writeObject("Hello");
}
```

- input потоки содержат методы для работой с порядком чтения данных. ```mark()``` и ```reset()``` возвращают поток на позицию ранее. Перед вызовом методов, нужно проверить их поддержку с помощью ```markSupported()```:
```java
public boolean markSupported()
public void void mark(int readLimit)
public void reset() throws IOException
public long skip(long n) throws IOException
```

- метод ```flush()``` заставляет все buffered байты записаться в файл назначения. После записи всего файла необязательно вызывать ```flush()```, так как ```close()``` делает это автоматически:
```java
// OutputStream and Writer
public void flush() throws IOException
```

- конструкторы ```FileInputStream```, ```FileOutputStream```:
```java
public FileInputStream(File file) throws FileNotFoundException
public FileInputStream(String name) throws FileNotFoundException
public FileOutputStream(File file) throws FileNotFoundException
public FileOutputStream(String name) throws FileNotFoundException
```

- также существует конструктор ```FileOutputStream```, принимающий флаг append в качестве параметра. При append ```== true``` поток будет записывать в конец файла.

- buffered потоки содержат некоторое количество улучшений производительности для работы с данными в памяти. Такие потоки могут обрабатывать много байтов и не обращаться к файловой системе для чтения/записи каждого байта. В силу организации данных в компьютере предпочтительнее использовать степень 2 в качестве размера буфера (как вариант - от 1024 до 65536).

- схожие конструкторы классов для работы с текстовыми данными:
```java
public FileReader(File file) throws FileNotFoundException
public FileReader(String name) throws FileNotFoundException
public FileWriter(File file) throws FileNotFoundException
public FileWriter(String name) throws FileNotFoundException
```

- ```Writer``` позволяет записывать строки, а buffered классы содержат методы ```readLine()``` и ```newLine()```:
```java
// Writer
public void write(String str) throws IOException
// BufferedReader
public String readLine() throws IOException
// BufferedWriter
public void newLine() throws IOException
```

## Сериализация

- сериализация - это процесс преобразования объектов в памяти в поток байтов. Десериализация - обратный процесс.

- сериализуемый объект должен наследовать маркерный интерфейс ```Serializable```. Любое поле, помеченное ключевым словом ```transient``` не будет сохраняться в поток при сериализации объекта.

- хорошей практикой считается использования константы ```serialVersionUID``` в каждом классе, реализующем ```Serializable```. При каждом изменении структуры класса это значение обновляется (например, увеличивается на 1).
```java
// Version 1
class Point implements Serializable {
    private static final long serialVersionUID = 1L;
    private int x;
    private int y;
}

// Version 2
class Point implements Serializable {
    private static final long serialVersionUID = 2L;
    private int xCoordinate;
    private int yCoordinate;
    private int zCoordinate;
}
```

- ```static``` поля (за исключением ```serialVersionUID``` не сериализуются). При десериализации ```transient``` данных им присваиваются значения по умолчанию (```0.0```, ```null``` и т.п.). Каждое поле сериализуемого класса должно быть:
    - либо сериализуемым
    - либо ```transient```
    - либо ```null``` в момент сериализации

- существуют потоки для сериализации/десериализации:
```java
public ObjectInputStream(InputStream in) throws IOException
public ObjectOutputStream(OutputStream out) throws IOException
```

- основные методы для чтения/записи объекта:
```java
// ObjectInputStream
public Object readObject() throws IOException, ClassNotFoundException
// ObjectOutputStream
public void writeObject(Object obj) throws IOException
```

- при десериализации объект, конструктор и instance initializer блоки сериализованного класса не вызываеются. Java вызовет конструктор без аргументов первого nonserializalbe родительского класса в иерархии наследования.

## Вывод данных

- ```PrintStream``` и ```PrintWriter``` не имеют сооветствующих аналогичных классов входных потоков. Их конструкторы:
```java
public PrintStream(OutputStream out)
public PrintWriter(Writer out)
public PrintStream(File file) throws FileNotFoundException
public PrintStream(String fileName) throws FileNotFoundException
public PrintWriter(File file) throws FileNotFoundException
public PrintWriter(String fileName) throws FileNotFoundException
public PrintWriter(OutputStream out)
```

- print stream классы включают методы ```print()```, ```println()```, ```format()```. Методы в print stream классах не бросают checked исключения.

- символы для метода ```format()```:
    - ```%s``` (любой тип, обычно ```String```)
    - ```%d``` (целые числа, ```int```, ```long```)
    - ```%f``` (числа с плавающей точкой, ```float```, ```double```)
    - ```%n``` (переход на новую строку)

- есть два объекта ```PrintStream``` для вывода информации пользователю: ```System.out``` и ```System.err``` (для вывода ошибок).

- ```System.in``` возвращает ```InputStream``` и используется для получения текстовых данных от пользователя (ввода):
```java
var reader = new BufferedReader(new InputStreamReader(System.in));
String userInput = reader.readLine();
System.out.println("You entered: " + userInput);
```

- закрывать системные потоки ввода/вывода не рекомендуется, так как потом их будет нельзя использовать.

<!-- console !-->


# Chapter 9. NIO.2

## Path

- ```java.nio.file.Path``` - аналог ```java.io.File```, представляет собой иерархический путь к файлу или директории в файловой системе. В отличие ```java.io.File```, ```Path``` поддерживать использование symbolic link (специальный файл в файловой системе, который является на другой файл или папку).

- получить объект ```Path``` можно с помощью фабричного метода:
```java
// Path factory method
public static Path of(String first, String… more)

// Examples
Path path1 = Path.of("pandas/cuddly.png");
Path path2 = Path.of("c:\\zooinfo\\November\\employees.txt");
Path path3 = Path.of("/home/zoodirectory");

// More
Path path1 = Path.of("pandas", "cuddly.png");
Path path2 = Path.of("c:", "zooinfo", "November", "employees.txt");
Path path3 = Path.of("/", "home", "zoodirectory");
```

- также для получения instance ```Path``` можно использовать фабричный метод у ```java.nio.file.Paths```:
```java
// Paths factory method
public static Path get(String first, String… more)
```

<!-- uri !-->

- можно получить instance ```FileSystem```, а из него получить ```Path```:
```java
// FileSystems factory method
public static FileSystem getDefault()

// FileSystem instance method
public Path getPath (String first, String… more)

// Examples
Path path1 = FileSystems.getDefault().getPath("pandas/cuddly.png");
Path path2 = FileSystems.getDefault()
 .getPath("c:\\zooinfo\\November\\employees.txt");
Path path3 = FileSystems.getDefault().getPath("/home/zoodirectory");
```

- есть возможность подключаться к удаленной (remote) файловой системе:
```java
FileSystem fileSystem = FileSystems.getFileSystem(
    new URI("http://www.selikoff.net"));
    Path path = fileSystem.getPath("duck.txt");
```

- ```Path``` содержит три метода для получения базовой информации о представлении пути и получении его частей:
```java
public String toString()
public int getNameCount()
public Path getName(int index)
```

- есть метод ```subpath()``` для получения подпути:
```java
public Path subpath(int beginIndex, int endIndex)
```

- можно проверить является ли путь абсолютным с помощью ```isAbsolute()```. Текущую рабочую директорию можно получить с помощью ```System.getProperty("user.dir")```:
```java
public boolean isAbsolute()
public Path toAbsolutePath()
```

- пути соединяются с помощью ```resolve()```:
```java
public Path resolve(Path other)
public Path resolve(String other)
```

- с помощью ```relativize()``` можно получить относительный путь относительно другого пути. При этом, оба пути должны быть одновременно либо абсолютными, либо относительными:
```java
var path1 = Path.of("fish.txt");
var path2 = Path.of("friendly/birds.txt");
System.out.println(path1.relativize(path2));
System.out.println(path2.relativize(path1));

// ../friendly/birds.txt
// ../../fish.txt
```

- ```normalize()``` позволяет избавляться от лишних символов в пути:
```java
var p1 = Path.of("./armadillo/../shells.txt");
System.out.println(p1.normalize()); // shells.txt
var p2 = Path.of("/cats/../panther/food");
System.out.println(p2.normalize()); // /panther/food
var p3 = Path.of("../../fish.txt");
System.out.println(p3.normalize()); // ../../fish.txt
```

- метод ```toRealPath()``` также избавляется от лишних символов в пути, а также бросает исключение в том случае, если путь не существует.

## NIO.2 Files

- метод для проверки на то, существует ли файл:
```java
public static boolean exists(Path path, LinkOption… options)
```

- метод для проверки на то, ссылаются ли два объекта ```Path``` на один и тот же файл (директорию):
```java
public static boolean isSameFile (Path path, Path path2) throws IOException
```

- метод ```createDirectory()``` создает директорию и бросает исключение, если она уже существует, или если пути до директории не существуют. ```createDirectories()``` создает директорию со всем несуществующими родительскими директориями, ведущими к пути:
```java
public static Path createDirectory (Path dir,
    FileAttribute<?>... attrs) throws IOException
public static Path createDirectories (Path dir,
    FileAttribute<?>... attrs) throws IOException
```

- метод для копирования файлов (директорий). Для перезаписи можно использовать опцию ```REPLACE_EXISTING```. Можно производить копирование, используя потоки:
```java
public static Path copy (Path source, Path target, CopyOption... options) throws IOException

// Example
Files.copy(Paths.get("book.txt"), Paths.get("movie.txt"),
    StandardCopyOption.REPLACE_EXISTING);

public static long copy (InputStream in, Path target, CopyOption… options) throws IOException
public static long copy (Path source, OutputStream out) throws IOException
```

- переименование или перемещение путей с помощью ```move()```:
```java
public static Path move (Path source, Path target, CopyOption... options) throws IOException

// Examples
Files.move(Path.of("c:\\zoo"), Path.of("c:\\zoo-new"));
Files.move(Path.of("c:\\user\\addresses.txt"),
    Path.of("c:\\zoo-new\\addresses2.txt"));
```

- метод для удаления. При этом, папка должна быть пустой:
```java
public static void delete (Path path) throws IOException
public static boolean deleteIfExists (Path path) throws IOException
```

- в ```Files``` существует метод для чтения всех строк файла в список:
```java
public static List<String> readAllLines(Path path) throws IOException
```

- методы для получения различных свойств файла:
```java
public static boolean isHidden(Path path) throws IOException
public static boolean isReadable(Path path)
public static boolean isWritable(Path path)
public static boolean isExecutable(Path path)
public static long size (Path path) throws IOException
public static FileTime getLastModifiedTime (Path path, LinkOption... options) throws IOException
```

- можно получать объект, содержащий атрибуты пути:
```java
var path = Paths.get("/turtles/sea.txt");
BasicFileAttributes data = Files.readAttributes(path,
 BasicFileAttributes.class);
System.out.println("Is a directory? " + data.isDirectory());
System.out.println("Is a regular file? " + data.isRegularFile());
System.out.println("Is a symbolic link? " + data.isSymbolicLink());
System.out.println("Size (in bytes): " + data.size());
System.out.println("Last modified: " + data.lastModifiedTime());
```

- можно изменять атрибуты (но не все) с помощью view объекта:
```java
// Read file attributes
var path = Paths.get("/turtles/sea.txt");
BasicFileAttributeView view = Files.getFileAttributeView(path,
    BasicFileAttributeView.class);
BasicFileAttributes attributes = view.readAttributes();
// Modify file last modified time
FileTime lastModifiedTime = FileTime.fromMillis(
    attributes.lastModifiedTime().toMillis() + 10_000);
view.setTimes(lastModifiedTime, null, null);
```

## Stream API

- получение содержимого директории:
```java
public static Stream<Path> list (Path dir) throws IOException
```

- deep копирование с использованием Stream API:
```java
public void copyPath(Path source, Path target) {
    try {
        Files.copy(source, target);
        if(Files.isDirectory(source))
            try (Stream<Path> s = Files.list(source)) {
                s.forEach(p -> copyPath(p,
                        target.resolve(p.getFileName())));
            }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

- ```Files``` содержит методы для обхода дерева папки, используя поиск в глубину (DFS):
```java
public static Stream<Path> walk(Path start,
    FileVisitOption... options) throws IOException
public static Stream<Path> walk(Path start, int maxDepth,
    FileVisitOption... options) throws IOException
```

- пример нахождения общего размера всех файлов в папке:
```java
private long getSize(Path p) {
    try {
        return Files.size(p);
    } catch (IOException e) {
        e.printStackTrace();
    }
    return 0L;
}
public long getPathSize(Path source) throws IOException {
    try (var s = Files.walk(source)) {
        return s.parallel()
                .filter(p -> !Files.isDirectory(p))
                .mapToLong(this::getSize)
                .sum();
    }
}
```

- также для поиска можно использовать метод ```find()```:
```java
public static Stream<Path> find (Path start,
    int maxDepth,
    BiPredicate<Path, BasicFileAttributes> matcher,
    FileVisitOption… options) throws IOException

// Example
Path path = Paths.get("/bigcats");
long minSize = 1_000;
try (var s = Files.find(path, 10,
        (p, a) -> a.isRegularFile()
                && p.toString().endsWith(".java")
                && a.size() > minSize)) {
    s.forEach(System.out::println);
}
```

- для построчного чтения файла используется ```lines()```. Метод похож на ```Files.readAllLines()```, однако преимущество ```lines()``` в том, что происходит lazy evaluation, и не требуется, чтобы файл хранился полностью в памяти:
```java
public static Stream<String> lines(Path path) throws IOException

// Example
Path path = Paths.get("/fish/sharks.log");
try (var s = Files.lines(path)) {
    s.forEach(System.out::println);
}
```

# Chapter 10. JDBC

# Chapter 11. Security