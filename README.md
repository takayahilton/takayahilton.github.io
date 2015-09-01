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


##install方法
[公式](https://www.rust-lang.org/install.html)のホームページいってinstallerとバイナリをダスンロード出来ますが、
```sh
$ curl -sSf https://static.rust-lang.org/rustup.sh | sh
```
でinstallが一番簡単
* cargoというパッケージビルドツールとrustコンパイラがインストール出来ます。
```sh
$ rustc main.rs
```
でコンパイルできる。



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

//代数的データ型
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


特殊なtraitを実装する事で演算子のオーバーロードが使える
```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {x: self.x + other.x, y: self.y + other.y}
    }
}

fn main() {
    let point = Point {x: 1, y: 0} + Point {x: 2, y: 3};
    println!("{:?}", point);
}

```




## Rustのポインタ
Rustのポインタにはlifetimeと所有権という概念があり、ヒープの解放などの処理はコンパイラが面倒を見てくれる。
```rust
//スタックへのポインタは返せない
fn fuge()->&i32{// missing lifetime specifier
  let x = 3;//スタックに変数を積む。
  &x;
}

fn main(){
  {
    let hoge = Box::new("hoge"); // 文字列をヒープに割り当てる。
  }//スコープの外に出ると解放する。
}
```


### 所有権の移動
Rustでは参照を関数に渡したり変数に代入すると所有権の移動が起こる。
```rust
fn boxed_int(i:Box<i32>){
  println!("{}", i);
}//ここでiは解放される

fn main(){
  let x = Box::new(3);
  boxed_int(x);
  println!("{}", x) // use of moved value: `x`
}
```
boxed_int関数に所有権が移ってしまったので、使用できない。
* 所有権の移動を行わず参照を渡したい場合は、所有権の借用を使う。


### 所有権の借用
&変数 で借用が使える
```rust
fn borrow_int(i:&i32){
  println!("{}",i);
}//iは解放されない

fn main(){
  let x = Box::new(3);
  borrow_int(&x);
  println!("{}", x);
}
```
貨しているだけなので　borrow_intが終了した後でも使用できる。


### 所有権があると何が嬉しいのか
* c++なんかだと突然死する可能性のあるコードをコンパイルエラーに出来る
```rust
fn main() {
    let mut x = Vec::new();
    x.push("hello");
    let y = &mut x[0]; //先頭の要素への参照
    x.push("world"); //　pushする事でyが無効なメモリを参照してしまう可能性がある。
    println!("{:?}",y);
}
```


コンパイルすると
```sh
error: cannot borrow `x` as mutable more than once at a time
```
ちゃんとコンパイルエラーになる


値をコピーするとコンパイルが通る
```rust
fn main() {
    let mut x = Vec::new();
    x.push("hello");
    let y = &mut x[0].clone();//クローンしている。
    x.push("world");
    println!("{:?}",y);
}
```




#まとめ
* RustはGCを使わずにメモリ安全にプログラミングが出来る。
* Rustは抽象的にプログラミング出来るっぽい
* 古いサンプルコードだとコンパイル通らなかったりする。　
* Rustのメモリ安全性の「寿命」や「所有権」の概念は難しい

理解できるまで時間がかかりそう。
