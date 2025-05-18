## 基本的なアクセス

**フォルダー `controller`の中で**、コントローラーのコードを書く。**機能によって、様々なクラスを定義する。**
```java
＠Controller
public class SessionController{
    @RequestMapping(path = "/doLogin", method = RequestMethod.GET) 
    public String doLoginGet(Integer userId) { 
        System.out.println("ユーザ ID:" + userId); 
        return "session/login"; 
    }
}
```
メソッドの中で、**パラメータと同じの名前の引数はパラメータの値を受ける**。

## Formクラス
引数は多すぎないように、**フォルダー`form`の中で**、`Form`クラスを定義する。
```java
public class LoginForm {

    private Integer userId; 
    private String password; 
 
    public Integer getUserId() { 
        return userId; 
    } 
    // ......
}
```
!!! note

    `setter()`必ず定義する。そうしないと、フィルードにパラメータの値は渡せない。また、フィルードとラメーターの名前は同じだ。


## Thymeleafを使う
```java
@RequestMapping("/items/findAll") 
public String showItemList(Model model) { 
    model.addAttribute("items", repository.findAll()); 
    return "items/item_list"; 
}
```
Model型の引数を使ったら、spring自動でModel型のオブジェクトを生成して、オブジェクトの中のデータはビューに入れて、Thymeleafはアクセスできる。

## データベースを使う

### 必要な構造

**フォルダー`entity`の中で**、エンティティクラスを実現する。
```java
@Entity
@Table(name = "items")
public class Item {
    @Id 
    @SequenceGenerator(name = "seq_items_gen", sequenceName = "seq_items", 
allocationSize = 1)
    @GeneratedValue(strategy = GenerationType.SEQUENCE, 
    generator = "seq_items_gen") 
    private Integer id;
	
    @Column 
    private String name; 
 
    @Column 
    private Integer price; 
    // .... ....setter getter
}

```
**フォルダー`repository`の中で**,リポジトリを定義する。
```java
public interface ItemRepository extends JpaRepository<Item, Integer> { 
    // Integerは主キーの型
	
	List<Item> findAllByOrderByPriceDesc(); 
	List<Item> findByPrice(Integer price);
	List<Item> findByNameAndPrice(String name, Integer price); 
	List<Item> findByNameContaining(String name);

}
```
ある規則に従って、インタフェースを宣言して、springは自動で実装できる。よく使われる規則は以下の通り：
```
[戻り値の型]　findAllByOrderBy[entity名][Asc|Desc](); //find all
[戻り値の型]　findBy[entity名](entity 値) // find entity
[戻り値の型]　findBy[entity1名]And[entity2名](entity1値,entity2値)
[戻り値の型]　findBy[entity名]Containing(検索値)　// like
```
もっと詳しく参考したら、[ここ](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html#jpa.query-methods.query-creation)。

### データの検索
`repository`を使うために、アノテーション`@Autowired`を付ける必要がある。
```java
@Controller
public class ItemController {
    @Autowired 
    ItemRepository repository; //beanになる、springで管理する
}
@RequestMapping("/items/findAll") 
public String showItemList(Model model) { 
    // 直接エンティティをビューに入れることは問題がある！！！！
    model.addAttribute("items", repository.findAll()); 
    return "items/item_list"; 
}
```
!!! note

    直接エンティティをビューに入れたら、問題がある。①見せたくないフィールドはビューに渡された。②結合度高い、もしエンティティのコードが変わったら、テンプレートのコードを変える必要がある。

問題を避けるように、JavaBeanを使う。**フォルダー `bean`の中で**、JavaBeanを定義する。エンティティはJavaBeanに変更して、ビューに入れる。
```java
//この二つのフィールドだけでテンプレートに渡す
public class ItemBean implements Serializable{
    private String name;
    private String price;
　　// こうコントラクターは役にたつ
    public ItemBean(Item item){
        this.name = item.getName();
        this.price = item.getPrice();
    } 
}
// Stream APIを使って、List<ItemBean>に変更する
List<Item> itemList = repository.findAll();
List<ItemBean> itemBeans = itemList.stream()
    .map(item -> new ItemBean(item))
    .collect(Collectors.toList());
// javaBeanはビューに入れる
model.addAttribute("items", itemBeans);
```
### データの更新
```java
public String createComplete(ItemForm form,Model model) { 
       Item item = new Item(); 
       // form->item 同じ名前フィールドをコーピする
       // idを除外する　
       BeanUtils.copyProperties(form, item, "id"); 
       item=repository.save(item); //データの更新
       ItemBean itemBean = new ItemBean(); 
       //ItemBean itemBean = new ItemBean(Item)
       BeanUtils.copyProperties(item,itemBean); 
       model.addAttribute("item", itemBean); 
       return "items/item"; 
}
```
## url中のパスパラメータ
```java
// メソッドのpriceとパス中のprice,名前は同じになる必要がある。そうしなければ
// PathVariable("price") Integer p で書く
@RequestMapping("/items/findByPrice/{price}") 
public String showItemListByPrice(@PathVariable Integer price, Model model) { 
	//......
} 
```
## アノテーション
-   `@Controller`:HTMLビュー（Thymeleafなど）を返す用途で使われる。クラスは `Bean`になる。
- `@RequestMapping`:URLパスとHTTPメソッドのマッピングを設定する。
- `@Autowired`: **Spring コンテナ内の Bean （一般的にBean、HttpSessionはBeanじゃないけど、@Autowiredが使える）を対象オブジェクト（フィールド、コンストラクタ、メソッドの引数など）に自動的に注入するために使用されます。**
    ```java
    // フィールドインジェクション
    @Autowired 
    ItemRepository repository; 
    
    ```
- `@PathVariable`:URLパスの中に含まれる変数部分をコントローラのメソッド引数にバインドするために使用する。
- `@Entity`:これは JPA のエンティティクラスであり、データベース内のテーブルに対応している。
- `@Table`:エンティティクラスに対応するデータベースのテーブル名を指定します（省略可能。省略した場合、クラス名がテーブル名として使用される）。
- `@Id`:エンティティクラスの中で、どのフィールドがデータベースの主キーであるかを示す。
- `@GeneratedValue`：**主キーの生成戦略**を指定するアノテーションで、通常は @Id と一緒に使用されます。一般的な戦略には、AUTO、IDENTITY、SEQUENCE、TABLE があります。
- `@SequenceGenerator`：**シーケンスジェネレータのアノテーションで**、@GeneratedValue と組み合わせて使用され、主キーの値がデータベース内のシーケンスによって生成されることを示します。
    ```java
    @SequenceGenerator(name = "item_seq", sequenceName = "item_seq", allocationSize = 1)
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "item_seq")
    ```
- `@Column`: エンティティクラスの属性とデータベースのカラムをマッピングする。


## Thymeleafの属性
-  `th:action`: フォームの送信先 URL を指定する。
    ```xml
    <form method="post" th:action="@{/items/create/complete}">
    ```
- `th:text`: ラベルの中の内容を設定する。
    ```xml
    <span th:text="${变量}"></span>
    <!-- イコール -->
    <span>变量的值</span> 
    ```
- `th:each`:リストや配列などのコレクションを繰り返し処理するための Thymeleaf 属性だ。for文見たいものだと思う。
    ```xml
    <!-- th:each は、繰り返しレンダリングできるタグ（例：<option>、<tr>、<li> など）でのみ使用でき -->
    <!-- selectに入れることはダメ -->
     
    <tr th:each="item : ${items}">
    <td th:text="${item.name}"></td>
    <td th:text="${item.price}"></td>
    </tr>
    <select name="itemdrop"> 
    <option th:each="item: ${items}" 
    th:value="${item.id}" th:text="${item.name}"></option> 
    </select> 

    ```

## 関連知識

### Session 

サイトにアクセスすると、サーバーが「この人は今こういう状態だな」と覚えておくための仕組みだ。springの中で`HttpSession`はメソッド引数依存性注入できる。
```java
@RequestMapping(path = "/doLoginOnSession",method = RequestMethod.POST) 
public String doLoginOnSession(LoginForm form, HttpSession session) { 
    if(form.getUserId() == 123) {
        session.setAttribute("userId", form.getUserId()); 
        return "redirect:/";
    }else { 
        return "session/login_on_session"; 
    } 
}
```
    
    















