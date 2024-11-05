---
title: Rust中使用Serde进行序列化和反序列化
description: 在rust中使用serde进行json, bson, yaml等格式的序列化/反序列化处理, 并且也可以自定义序列化/反序列化的方式
date: 2022-06-03 12:13:26
categories:
  - Coding
tags:
  - Rust
  - Serde
  - 序列化
  - 反序列化
  - Json
  - Yaml
---

# Rust 中使用 Serde 进行序列化和反序列化

在 rust 中使用 serde 进行 json, bson, yaml 等格式的序列化/反序列化处理, 并且也可以自定义序列化/反序列化的方式

## 基础

### 安装

使用 Serde 之前需要生命依赖, 在 Cargo.toml 中声明:

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0" # 处理json时可以生命 serde_json
```

features = ["derive"]表明使用 Serde 的派生宏来自动生成序列化/反序列化代码

### 序列化

要用 Serde 进行序列化, 需要让 struct 实现 serde::Serialize trait, 如:

```rust
#[derive(Serialize)]
struct People {
  name: string
  age: u32
}
```

然后使用 serde_json 库将 People 序列化为 json 字符串

```rust
use serde_json

let people = People {
  name: "Pele".to_owned(),
  age: 84,
};
let json = serde_json::to_string(&people).unwrap();
println!("{}", json); // {"name":"Pele","age":84}
```

### 反序列化

要用 Serde 进行反序列化, 需要让 struct 实现 serde::Deserialize trait, 如:

```rust
#[derive(Deserialize, Debug)]
struct People {
  name: string
  age: u32
}
```

然后使用 serde_json 库将 json 字符串反序列化为 People

```rust
use serde_json

let json = r#"{"name":"Pele","age":84}"#;
let people = serde_json::from_str(json).unwrap();
println!("{:?}", people); // People { name: "Pele", age: 84 }
```

## 进阶

### 自定义

Serde 可以自定义序列化/反序列化方式, 如将 People 结构体在序列化时 name 值大写, 反序列化时转为小写:

```rust
use serde::{Serialize, Deserialize, Serializer, Deserializer};

#[derive(Serialize, Deserialize, Debug)]
struct People {
    #[serde(serialize_with = "serialize_name", deserialize_with = "deserialize_name")]
    name: String,
    age: u32,
}

fn serialize_name<S>(name: &String, serializer: S) -> Result<S::Ok, S::Error>
where
    S: Serializer,
{
    serializer.serialize_str(&name.to_uppercase())
}

fn deserialize_name<'de, D>(deserializer: D) -> Result<String, D::Error>
where
    D: Deserializer<'de>,
{
    let name = String::deserialize(deserializer)?;
    Ok(name.to_lowercase())
}
```

在该结构体中, 使用 #[serde(serialize_with = "serialize_name", deserialize_with = "deserialize_name")] 指定了自定义序列化/反序列化方式

### 枚举

Serde 同样支持枚举的序列化/反序列化

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
#[serde(tag = "type")]
enum People {
    American { name: String, age: u32 },
    Japanese { name: String, age: u32 },
}
```

在处理时, 需要用 #[serde(tag = "type")] 指定枚举类型的标签

```rust
use serde_json;

let american = People::American { name: "Tod".to_owned(), age: 24 };
let json = serde_json::to_string(&american).unwrap();
println!("{}", json); // {"type":"American","name":"Tod","age":24}

let json = r#"{"type":"American","name":"Tod","age":24}"#;
let american: People = serde_json::from_str(json).unwrap();
println!("{:?}", american); // American { name: "Tod", age: 24 }
```

### 结构体中的 Option

Serde 支持序列化/反序列化结构体中的 Option 类型, 例如:

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct People {
    #[serde(skip_serializing_if = "Option::is_none")]
    name: Option<String>,
    age: u32,
}
```

在处理 Option 类型时, 需要指定 #[serde(skip_serializing_if = "Option::is_none")], 当 Option 值为 None 时, 不进行序列化

```rust
use serde_json;

let animal = Animal { name: Some("Tod".to_owned()), age: 3 };
let json = serde_json::to_string(&animal).unwrap();
println!("{}", json); // {"name":"Tod","age":3}

let animal = Animal { name: None, age: 3 };
let json = serde_json::to_string(&animal).unwrap();
println!("{}", json); // {"age":3}

let json = r#"{"age":3}"#;
let animal: Animal = serde_json::from_str(json).unwrap();
println!("{:?}", animal); // Animal { name: None, age: 3 }
```

### 结构体中的 Vec

Serde 支持序列化/反序列化结构体中的 Vec 类型, 例如:

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Family {
  members: Vec<People>,
}
```

Serde 会自动处理 Vec 类型

```rust
use serde_json;

let family = Family { members: vec![
    People { name: "Tod".to_owned(), age: 3 },
    People { name: "Curt".to_owned(), age: 2 },
] };
let json = serde_json::to_string(&family).unwrap();
println!("{}", json); // {"members":[{"name":"Tod","age":3},{"name":"Curt","age":2}]}

let json = r#"{"members":[{"name":"Tod","age":3},{"name":"Curt","age":2}]}"#;
let family: Family = serde_json::from_str(json).unwrap();
println!("{:?}", family); // Family { members: [People { name: "Tod", age: 3 }, People { name: "Curt", age: 2 }] }
```

### 结构体中的 HashMap

Serde 支持序列化/反序列化结构体中的 HashMap 类型, 例如:

```rust
use std::collections::HashMap;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Family {
    members: HashMap<String, People>,
}
```

Serde 会自动处理 HashMap 类型

```rust
use serde_json;

let mut members = HashMap::new();
members.insert("Tod".to_owned(), People { name: "Tod".to_owned(), age: 3 });
members.insert("Curt".to_owned(), People { name: "Curt".to_owned(), age: 2 });
let family = Family { members };
let json = serde_json::to_string(&family).unwrap();
println!("{}", json); // {"members":{"Curt":{"name":"Curt","age":2},"Tod":{"name":"Tod","age":3}}}

let json = r#"{"members":{"Curt":{"name":"Curt","age":2},"Tod":{"name":"Tod","age":3}}}"#;
let family: Family = serde_json::from_str(json).unwrap();
println!("{:?}", family); // Family { members: {"Tod": People { name: "Tod", age: 3 }, "Curt": People { name: "Curt", age: 2 }} }
```
