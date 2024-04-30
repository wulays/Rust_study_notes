#### HashMap

和动态数组一样，`HashMap` 也是标准库提供的集合类型，与动态数组不同的是 `HashMap` 中存储的是 键值对 `key: value` 的形式，当希望通过 `key` 来查询值得时候就很有用了

和 `Vec` 的创建方法类似，可以使用 `new` 创建，然后使用 `insert` 来插入

```rust
use std::collections::HashMap;

let mut foo = HashMap::new();

foo.insert("a", 1);
foo.insert("b", 2);
```

和 `Vec` 一样的，`key` 必须要拥有同样的类型，`value` 也是一样。这个时候 `HashMap` 的类型就是 `HashMap<&str, i32>` ，因为 `HashMap` 并没有被包含到 `prelude` 中所有需要手动引入。

> 跟 `Vec` 一样，如果预先知道要存储的 `KV` 对个数，可以使用 `HashMap::with_capacity(capacity)` 创建指定大小的 `HashMap`，避免频繁的内存分配和拷贝，提升性能。

#### 使用迭代器和 collect 创建

有时候需要从已有的数据结构中来获取到对应的数据创建 `HashMap`

```rust
let foo = vec![
	("张三".to_string(), 200),
	("李四".to_string(), 50),
	("王五".to_string(), 100)
];

let mut bar = HashMap::new();
for item in &foo {
	bar.insert(&item.0, item.1)
}
```

遍历列表，然后将元组插入 `HashMap` 中，不过 Rust 还提供了一个精妙的方法，将 `Vec` 转为迭代器，然后通过 `collect` 方法来收集迭代器中的元素，转为 `HashMap`

```rust
// 这里可以写具体类型，也可以写占位符 _ 让编译器帮组我们来推断
let bar: HashMap<_, _> = foo.into_iter().collect();
```

这里编译器会推导出来要收集一个 `HashMap` 集合，先使用 `into_iter()` 转为了迭代器，然后使用 `collect()` 来将迭代器的元素进行收集，这个时候会将第一个元素作为 `key` 第二个作为 `value`

> - `.iter()`：借用集合中每个元素的不可变引用。
> - `.iter_mut()`：借用集合中每个元素的可变引用。
> - `.into_iter()`：取得集合中每个元素的所有权。

所有权和别的类型一样的，如果实现了 `Copy` 就复制，没有实现就转移，同时也要注意，如果引用类型放入了 `HashMap` 中，确保这个引用的类型生命周期至少和 `HashMap` 一样

```rust
let name = String::from("test");
foo.insert(&name, 18);

std::mem::drop(name);

println("{:?}", foo);
```

这里就会报错了，因为获取到了 `name` 的引用并插入到了 `HashMap` 中，然后又删除了 `name` 字符串

如果要获取 `HashMap` 中的元素可以使用 `get` 方法，和 `Vec` 一样获取到的是一个 `Option<T>` 的引用类型如果查询不到返回 `None` 

```rust
foo.get("test");
```

如果想要直接获取到值的类型而不是引用类型，可以使用 `copied` 方法，会返回一个新的 `Option<T>` 或者 `Result<T>` 这个方法只适用于实现了 `Copy` 特征的类型

如果还想直接获取到值，而不去匹配可以使用 `unwrap_or` 返回的是 `Some<T>` 等就获取到 `T` 如果是 `None` 就会返回传入的参数默认值

```rust
let a = foo.get("test").copied().unwrap_or(0);
```

这里就是先拷贝如果有值就取值没有则默认为 0

使用 `insert` 是可以插入新值的如果已经有了则覆盖这个值，使用 `get` 来获取值，还能使用 `entry` 查询一个值是否存在返回 `Entry` 枚举，键已存在（`Occupied`），或键不存在（`Vacant`），再使用 `or_insert` 来对不存在的值进行插入，`or_insert` 返回一个值的可变引用，如果没有插入返回原来的值，返回新插入的值

```rust
let a = foo.entry("foo").or_insert(100);

*a += 10
```

可以根据拿到的可变引用来进行修改

#### 哈希函数

为什么叫哈希表，这是因为要实现 `key` 和 `value` 的一一对应，意味着要对两个 `key` 进行相等性比较。避免错误的映射。因此一个类型能不能作为 `key` 关键在于能否进行相等比较，或者说实现了 `std::cmp::Eq` 特征

> f32 和 f64 浮点数，没有实现 `std::cmp::Eq` 特征，因此不可以用作 `HashMap` 的 `Key`。

那如果要存储一个复杂类型作为 `key` 如何进行查询比较和存储。这个时候就需要用到哈希函数了，将 `key` 计算为哈希值，然后使用哈希值来进行存储和访问查询等

如何保证不同的 `key` 哈希后的值不相同，这个时候就涉及到安全性和性能的取舍了，如果要追求安全性减少冲突等，就要使用密码学安全的哈希函数，`HashMap` 就是使用了这样的哈希函数。如果要追求高性能就想要使用没有那么安全的算法

##### 高性能三方库

可以再 `crates.io` 上去寻找哈希函数的实现

```rust
use std::hash::BuildHasherDefault;
use std::collections::HashMap;
// 引入第三方的哈希函数
use twox_hash::XxHash64;


// 指定HashMap使用第三方的哈希函数XxHash64
let mut hash: HashMap<_, _, BuildHasherDefault<XxHash64>> = Default::default();
hash.insert(42, "the answer");
assert_eq!(hash.get(&42), Some(&"the answer"));
```

在 Rust 中，`HashMap<K, V, S>` 类型有三个泛型参数：
- `K`: 表示 key 的类型。
- `V`: 表示 value 的类型。
- `S`：表示用于散列的构建器的类型，默认为 `RandomState` 类型，这是一个由 Rust 标准库自带的随机化散列构建器，提供了一定程度的抵抗散列碰撞攻击的能力。内部使用的是 `SipHash` 算法

简单说 `HashMap` 会调用 `T::default` ，然后这里传入了 `BuildHasherDefault<XxHash64>` 指定了类型为 `XxHash64` 也就是到时候 `default` 会去调用 `XxHash64::default()` 这个是在 Rust 类型系统在编译时静态决定的，间接的调用了

> 目前，`HashMap` 使用的哈希函数是 `SipHash`，它的性能不是很高，但是安全性很高。`SipHash` 在中等大小的 `Key` 上，性能相当不错，但是对于小型的 `Key` （例如整数）或者大型 `Key` （例如字符串）来说，性能还是不够好