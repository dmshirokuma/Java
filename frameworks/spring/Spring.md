# 主な構成要素

- IoC（Inversion of Control）
  - ソフトウェアの原則（SOLID）の依存性の逆転を目指している
- DI(Dependency Injection)
  - IoCを実現するための実装（別途定義された情報に基づいてデータを注入する）
  - これによってコンポーネント間を疎結合に保つことができる
- BEANs
  - Springによってインスタンス化されたPOJO
- Context
- SpEL
- IoC Container
  - Beansのライフサイクルやスコープ等を管理する
  - 依存性の注入は実際にIoC Containerによって行われる
  - 実装タイプは主に以下の２種類
    - BeanFactory(Basic)
    - ApplicationContext(Advanced)

## Annotation

### @Bean

Springで管理するインスタンスやメソッドに付与する（メソッドの場合でも名詞系にする必要あり）

### @Configuration

Springがクラス内の全てのコンテンツをスキャンし、@Beanの付与されたデータを認識する

```Java
@Configuration
public class ProjectConfig {

    @Bean`
    Vehicle vehicle() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name");
        return veh;
    }
}
```

以下のようにApplicationContextからBeanを取得できる

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //Bean取得
        var veh = context.getBean(Vehicle.class);
        System.out.println("Vehicle name from Spring Context is :" + veh.getName());
    }
}
```

実行結果

```Text
Vehicle name from Spring Context is :Vehicle Name
String value from Spring Context is :Hello World
Integer value from Spring Context is :16
```

ただし、以下のように同じクラスを返却するBeanが２つ以上あるとエラーが発生する

```Java
@Configuration
public class ProjectConfig {

    @Bean
    Vehicle vehicle() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name");
        return veh;
    }

    @Bean
    Vehicle vehicle2() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name2");
        return veh;
    }
}
```

上記のような場合は以下に示す通り、getBeanの第一引数に識別子（この場合はメソッド名）を指定する。

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //識別子を指定してBean取得
        var veh = context.getBean("vehicle2", Vehicle.class);
        System.out.println("Vehicle name from Spring Context is :" + veh.getName());
    }
}
```

実行結果

```Text
Vehicle name from Spring Context is :Vehicle Name2
```

もしくは以下のようにBeanに直接名称を付与することによって名前指定でBeanを取得することも可能。

```Java
@Configuration
public class ProjectConfig {

    @Bean(value = "vehicle1bean") //valueで名称を付与する
    Vehicle vehicle() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name");
        return veh;
    }

    @Bean(value = "vehicle2bean")
    Vehicle vehicle2() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name2");
        return veh;
    }
}

```

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //Bean取得
        var veh = context.getBean("vehicle1bean", Vehicle.class);
        System.out.println("Vehicle1 name from Spring Context is :" + veh.getName());

        var veh2 = context.getBean("vehicle2bean", Vehicle.class);
        System.out.println("Vehicle2 name from Spring Context is :" + veh2.getName());
    }
}
```

実行結果

```Text
Vehicle1 name from Spring Context is :Vehicle Name
Vehicle2 name from Spring Context is :Vehicle Name2
```

### @Primary

同クラスを返却するBeanの中でもデフォルトを指定することができる。  
@Primaryアノテーションが複数に付与されているとエラーとなるので注意。

```Java
@Configuration
public class ProjectConfig {

    @Bean(value = "vehicle1bean")
    Vehicle vehicle() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name");
        return veh;
    }

    @Bean(value = "vehicle2bean")
    Vehicle vehicle2() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name2");
        return veh;
    }

    @Primary
    @Bean(value = "vehicle3bean")
    Vehicle vehicle3() {
        var veh = new Vehicle();
        veh.setName("Vehicle Name3");
        return veh;
    }
}
```

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //Bean取得
        var veh = context.getBean(Vehicle.class);
        System.out.println("Vehicle name from Spring Context is :" + veh.getName());
    }
}
```

実行結果

```Text
Vehicle name from Spring Context is :Vehicle Name3
```

### @Component @ComponentScan

@ComponentアノテーションをJavaのPOJOに付与すると自動的にPOJOをBeanとしてSpringが管理してくれる。
管理するパッケージは@ComponentScanのbasePackagesにて指定する。
指定したパッケージ配下の@Componentを全てスキャンしてくれるため、
都度生成メソッドを作って@Beanアノテーションを付与する必要がなくなる。

```Java
package com.example.beans;

import org.springframework.stereotype.Component;

@Component
public class Vehicle {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void Greet() {
        System.out.println("Hello");
    }
}
```

```Java
@Configuration
@ComponentScan(basePackages = "com.example.beans") //スキャン対象とするパッケージを指定
public class ProjectConfig {
}
```

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //Bean取得
        var veh = context.getBean(Vehicle.class);
        veh.Greet();
    }
}
```

#### 複数パッケージのスキャン

@ComponentScanで複数のパッケージをスキャンしたい場合は配列で指定することが可能

```Java
@Configuration
@ComponentScan(basePackages = {"com.example.beans", "com.example.beans2"})
public class ProjectConfig {
}
```

### @Controller @Service @Repository @Component

全てJavaのクラスをBeanとしてSpringが管理するために付与するアノテーションだが  
可読性向上のためそれぞれ以下に付与する。  
（実際@Controller @Service @Repositoryそれぞれの内部実装を見ても@Componentに対してのエイリアスとなっている。）

#### @Controller

コントローラ層のクラス（Requestのマッピングなどを行うクラス）に付与する。

#### @Service

サービス層のクラス（ビジネスロジックや他APIの呼び出しなどを行うクラス）に付与する。

#### @Repository

データアクセス層のクラス（DBなどのデータストアに対する処理行うクラス）に付与する。

#### @Component

上記のどれにも当てはまらないクラス（DTOなどのクラス）に付与する。

### @PostConstruct

@Beanと異なり、@Component単体ではBeanに対する初期化処理を行うことはできないが、  
@Componentを付与したクラス内の初期化メソッドに@PostConstructを付与することによって  
@Componentを付与したクラスの初期化を行うことができる。

ただし、@PostConstuctを使用する場合はMaven等で以下のパッケージ導入が必要となる。

```xml
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```

```Java
@Component
public class Vehicle {

    private String name;

    @PostConstruct //初期化処理
    public void Initialize() {
        this.name = "Vehicle name";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //Bean取得
        var veh = context.getBean(Vehicle.class);
        System.out.println("Vehicle name from Spring Context is :" + veh.getName());
    }
}
```

実行結果

```Text
Vehicle name from Spring Context is :Vehicle name
```

### @PreDestroy

@PreDestroyアノテーションを使用することによって  
ApplicationContextを破棄する直前にBeanに対して処理を行うことができる。  
通常はアプリケーションのシャットダウン時などに処理を行いたい場合に使用する。  
ファイルやデータベースなどのIOリソースを閉じる必要がある場合など。

```Java
@Component
public class Vehicle {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @PreDestroy //ApplicationContext破棄の直前に実行される
    public void destroy() {
        System.out.println("Destroy Process");
    }
}
```

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //ApplicationContext破棄
        context.close();
    }
}
```

実行結果

```Text
Destroy Process
```

### 独自にBeanをApplicationContextに設定する

ApplicationContextのregisterBeanを使用することによって可能。

```Java
public class Vehicle {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```Java
public class Example1 {
    public static void main(String[] args) {

        //ApplicationContext生成
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);

        //サプライヤーを定義
        Supplier<Vehicle> name1Supplier = () -> {
            var vehicle = new Vehicle();
            vehicle.setName("Vehicle Name1");
            return vehicle;
        };

        //Bean名、クラス、サプライヤーをそれぞれ渡してBeanをApplicationContextに登録する
        context.registerBean("Name1Vehicle", Vehicle.class, name1Supplier);

        var bean = (Vehicle)context.getBean("Name1Vehicle");
        System.out.println("Vehicle name from Spring Context is :" + bean.getName());
    }
}
```

実行結果

```Text
Vehicle name from Spring Context is :Vehicle Name1
```
