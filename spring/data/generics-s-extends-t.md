# ジェネリクス `<S extends T>`

## 概要
`<S extends T>`は、CrudRepositoryのsaveメソッドで使われるジェネリクスで、エンティティの継承関係を保持したまま型安全に保存・返却を行う。

## 使用場面
- エンティティの継承階層がある場合（例: Animal → Dog, Cat）
- サブクラスの型情報を失わずにCRUD操作を行いたい場合
- 型安全なリポジトリ操作を実現したい場合

## コード
```java
// CrudRepository の save メソッド定義
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S entity);
    //  ↑         ↑
    //  |         戻り値も S 型
    //  S は T のサブクラス（または T 自身）
}
```

```java
// 継承がある場合の例
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Animal {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}

@Entity
public class Dog extends Animal {
    private String breed;  // 犬種

    public String getBreed() { return breed; }
}

@Entity
public class Cat extends Animal {
    private boolean indoor;  // 室内飼いかどうか

    public boolean isIndoor() { return indoor; }
}
```

```java
// <S extends T> の恩恵
public interface AnimalRepository extends CrudRepository<Animal, Long> {
    // save メソッドは <S extends T> S save(S entity) として定義されている
}

@Service
public class AnimalService {
    private final AnimalRepository animalRepository;

    public void example() {
        Dog dog = new Dog();
        dog.setName("Pochi");
        dog.setBreed("Shiba");

        // <S extends T> のおかげで Dog 型のまま返ってくる
        Dog savedDog = animalRepository.save(dog);
        String breed = savedDog.getBreed();  // ✅ Dog のメソッドが使える

        Cat cat = new Cat();
        cat.setName("Tama");
        cat.setIndoor(true);

        // Cat 型のまま返ってくる
        Cat savedCat = animalRepository.save(cat);
        boolean indoor = savedCat.isIndoor();  // ✅ Cat のメソッドが使える
    }
}
```

```java
// もし <S extends T> ではなく、単に T だったら...
public interface BadCrudRepository<T, ID> {
    T save(T entity);  // 戻り値が T 固定
}

// こうなってしまう
Animal savedDog = animalRepository.save(dog);  // Animal 型で返ってくる
// savedDog.getBreed();  // ❌ コンパイルエラー！Animal に getBreed() はない

// キャストが必要になる（型安全でない）
Dog savedDog = (Dog) animalRepository.save(dog);  // 😢
```

## 説明
### `<S extends T>` の意味
- **S**: saveメソッドに渡される実際の型
- **extends T**: SはTのサブクラス（またはT自身）でなければならない
- **戻り値もS**: 入力と同じ型で返却される

### なぜ必要？
```
継承階層:
Animal (T)
├── Dog (S)  ← save(Dog) → Dog が返る
└── Cat (S)  ← save(Cat) → Cat が返る
```

### 継承を使わない場合
- SとTが同じになる
- 「同じ型が返る」という単純な理解でOK

```java
// User に継承がない場合
public interface UserRepository extends CrudRepository<User, Long> {}

User user = new User();
User savedUser = userRepository.save(user);  // S = T = User
```

## 参考
- [Java Generics - Upper Bounded Wildcards](https://docs.oracle.com/javase/tutorial/java/generics/upperBounded.html)
- [CrudRepository JavaDoc](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)

## 関連スニペット
- [Repository<T, ID>の型引数](./repository-type-parameters.md)
- [CrudRepositoryの基本メソッド](./crud-repository-methods.md)

## 作成日
2026-01-29

## タグ
#spring #spring-data #generics #inheritance #type-safety
