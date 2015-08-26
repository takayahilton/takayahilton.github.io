# Rust触ってみた
________



### Rustとは

Mozillaが開発中のシステム言語
* 今年の5月に1.0がリリースされた
* 構文はCに近い
* 型推論や代数的データ型、パターンマッチ、型クラスなどをサポート
* __ポインタ__が使える GCは積んでない
* メモリ安全（ここら辺の概念が一番難しく感じる）

色々尖ってる



## hello world
```rust
fn main(){
  println!("hello world")
}
```


printlnは分かるけど後ろについている !は何？


答えはマクロ
hello worldからマクロ


```rust
macro_rules! println {
    ($fmt:expr) => (print!(concat!($fmt, "\n")));
    ($fmt:expr, $($arg:tt)*) => (print!(concat!($fmt, "\n"), $($arg)*));
}
```
こんな感じに展開される　マクロで定義されたものは!マークがつく。
macro_rules!はマクロ定義の構文らしい



## 基本構文

#### 式
```rust
let x = 3; //再代入不可
let mut x = 3; //再代入できる
```

#### 関数定義
```rust
fn add(a:int, b:i32)->i32{
  a + b
}
```
* 関数定義の時に型は推論できない。(戻り値の型も省略できない)
* 最後の式が戻り値


### データ型
```rust
struct Point {
    x: i32,
    y: i32,
}

enum Animal{
  Cat,
  Dog,
}

enum Option<T> {
    Some(T),
    None,
}
```


### パターンマッチ
```rust
let x = Some("fuge");

match x {
  None => println!("None!"),
  Some("hoge") => println!("hoge"),
  Some("fuge") => println!("fuge"), // match!
}
```


### クロージャ
```rust
let sum = (0..101).fold(0, |sum, n| sum + n);
println!("{}", sum);
```


### trait
```rust
trait Foo {
    fn method(&self);
}
```


rustのトレイトは型クラスのように扱える。
実装を書くことでFooとして型制約を書ける。
(selfを引数を取るとmethodのように書けるのがpythonっぽい)
```rust
impl Foo for String {
    fn method(&self){println!("{}", *self)}
}

fn foo<T:Foo>(t:T){
    t.method();
}

foo("foo".to_string)//OK!
foo(3) // error: the trait `Foo` is not implemented for the type `_`
```




## Rustのポインタ
Rustのポインタにはlifetimeと所有権という概念があり、ヒープの解放などの処理はコンパイラが面倒を見てくれる。
```rust
fn fuge()->&i32{// missing lifetime specifier
  let x = 3;//スタックに変数を積む。
  &x;
}

fn main(){
  {
    let hoge = Box::new("hoge"); // 文字列をヒープに割り当てる。
  }//スコープの外に出るとコンパイラが解放する。
}
```

＃## 所有権の移動
```rust
fn boxed_int(i:Box<i32>){
  println!("{}", i);
}

fn main(){
  let x = Box::new(3);
  boxed_int(x);
  println!("{}", x) //use of moved value: `x`
}
```
boxed_int関数に所有権が移ってしまったので、使用できない。


### 所有権の借用
```rust
fn borrow_int(i:&i32){
  println!("{}",i);
}

fn main(){
  let x = Box::new(3);
  borrow_int(&x);
  println!("{}", x);
}
```
貨しているだけなので使用できる。
