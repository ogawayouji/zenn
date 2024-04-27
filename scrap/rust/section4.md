# 第四章　ジェネリック型

## そもそもジェネリック型とは

structやenumを**部分的に**実装できるようにする型**。**

（例：型を柔軟に当てたい、特殊な状況を型に含めたい、など。）

※　ジェネリック（Generic）は、「一般的な」や「共通」の意味を持つ英単語

```rust
struct GenericStruct <T> {
	item: T,
}

fn main() {
	// turbofish演算子を使った明示的な型指定の書き方
	let genericStruct1 = GenericStruct::<i16> { item: 10 };

	// rustの型推論は、ジェネリック型も推測してくれる
	let genericStruct2 = GenericStruct { item: 10 }

	// やろうと思えば入れ子も可能（バグの温床になりかねないのでできるだけ避ける）
	let genericStruct3 = GenericStruct {
		item: GenericStruct {
			item: 10
		}
	}
}
```

## Rustには「null」がない

他の言語ではよく用いられる、項目や値がないことを示すnullがRustにはない。

代わりに、**None**を用いる。

# 特徴的なジェネリック型

## Option

ジェネリックな**Enum型**の一つ。

上記で述べた、値がないことを表現する**None**と必ず存在する値であることを示す**Some**で構成される。

```rust
enum Option<T> {
	None,
	Some(T),
}
```

```rust
fn main() {
	// SomeおよびNoneはOption固有の表現で、省略して記述することが可能
	let x = Some(100);
	let y = None;

	// match文で分岐可能
	let z = match x {
		Some(i) => i,
		None => -1
	}
}
```

## Result

失敗する可能性のある値を返すことができるジェネリックなEnum型。

```rust
enum Result<T, E? {
	Ok(T),
	Err(E),
}
```

```rust
fn sample_func(i: i32) -> Result<i32, String> {
	if i < 100 {
		Ok(i)
	} else {
		Err(String::from("100未満ではない"))
	}
}

fn main() {
	//
	let result = sample_func(99);

	// こちらもmatchで分解可能
	match result {
		Ok(i) => println!("成功 {}",i),
		Err(e) => println!("失敗"),
	}
}
```

<aside>
💡 main関数もResultを返すことができる。

</aside>

### エラーハンドリング

Result型はTypescriptなどの非同期処理と同様にエラーハンドリングができる。

エラーハンドリング用のメソッドもデフォルトで定義されている。

```rust
// 以下は等価
do_something_that_might_fail()?

match do_something_that_might_fail() {
	Ok(v) =. v,
	Err(e) => return Err(e),
}
```

## ベクタ型（コレクション型）

ジェネリック型には、コレクション型（複数の値を取り扱うのに有用な型）を含む。

他言語では配列型や辞書型、集合型がある。

**Vec（ベクタ）**は可変サイズのリストである。

iter()メソッドでベクタのイテレータを作成可能。

→　for文などでベクタを回せるようになる

```rust
fn main() {
	// pushするなら、Vecは可変でなければならない（ベクタのメモリの長さが変化する可能性があるため）
	let mut sample_vec = Vec::new();
	sample_vec.push(1);
	sample_vec.push(2);

	// マクロを使った定義方法。
	let sample_vec2 = vec![String::from("a"), String::from("b")];
}
```

ベクタはヒープメモリに作成される。最初はデフォルト長の長さの領域が確保される。

デフォルト長で足りなくなった場合、データを再割り当てする。

## 値を簡単に取り出す方法

Option型のSomeと、Result型のOkを即座に取り出したい場合、**unwrap**メソッド簡単に（分岐構文を使わずに）取り出すことが可能。

ただし、NoneやErrを考慮しないメソッドのため、**失敗することがある**。

もし失敗する場合は、unwrapメソッドはpanic!（プログラム失敗のメッセージを表示し、プログラムを終了させる処理）を実行する。

```rust
// 以下は等価, Resultも同様

some_option.unwrap()

match some_option {
	Some(v) => v,
	None => panic!("エラーメッセージ"),
}
```