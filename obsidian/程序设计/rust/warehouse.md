# 仓库管理系统

## 特性需求
### 安全性
安全性需要后端基础设施完备且前后端配合,并且合理部署
详细说明见后端重点

### 易于部署
目标是通过单一二进制文件在任何主流平台简单完成部署.包括嵌入式数据库,数据迁移,管理员用户生成等初始化流程自动化完成,通过环境变量方便配置,单一指令快速启动

### 跨平台
在开发中我主要使用linux环境,x86-64架构,但通过在不同平台和架构的构建来实现跨平台和多架构的兼容.这也使得程序的各个部分集成程度很高

## 应用程序架构

- 前端使用React jsx
- 后端使用Rust
- 采用嵌入式数据库,Diesel 与 SQLite 结合
- 由Rust提供静态文件服务,使得前后端分离设计但是部署时统一
- web框架由Rocket提供
- 去中心化的仓库管理系统,各个仓库节点独立运行并相互协作

## 功能需求

- 入库
- 出库
- 货品查询(多仓库联动)
- 货品分类
- 多管理员
- 不同仓库间货品调度(通过不同服务端联动实现,一个服务对应一个仓库)


# 后端

## 依赖

```toml
[package]

name = "app1"

version = "0.1.0"

edition = "2021"

  

[dependencies]
# 用于数据库的对象关系映射

diesel = { version = "^2.1", features = ["sqlite", "r2d2", "chrono"] }
# 用于初始化环境变量

dotenvy = "^0.15"

# 用于将数据结构序列化和反序列化
serde = { version = "^1.0", features = ["derive"] }
# 用于数据结构序列化和反序列化为json
serde_json = "^1.0"

# 处理时间格式的库
chrono = { version = "^0.4", features = ["serde"] }
# web框架
rocket = { version = "^0.5", features = ["json"] }
# 用于为web服务附加数据库连接池
rocket_sync_db_pools = { version = "^0.1", features = ["diesel_sqlite_pool"] }

# 用于web框架处理跨域资源访问
rocket_cors = "0.6.0"

  

# rusqlite = "^0.31"
# 用于生产随机字符串
rand = "^0.8"
# 用于构建数据库
libsqlite3-sys = { version = "^0.28", features = ["bundled"] }

  
# 用于对函数进行同步或异步处理
async-std = "^1"

# 用于异步I/O和跨进程通信的抽象
futures = "^0.3"

# 用于生成uuid
uuid = { version = "1", features = ["v4"] }


  
# 用于构建p2p网络
libp2p = "0.46.1"

# 用于提供异步运行时
tokio = { version = "1.19.2", features = ["full"] }

tokio-util = { version = "0.7.11", features = ["compat", "io"] }

# 日志记录器
env_logger = "^0.11"

log = "^0.4"

log4rs = "^1.3"

# 提供数据结构序列化与反序列化的yaml支持

serde_yaml = "0.9.34+deprecated"

# 提供为路由使用jwt的相关支持 

jsonwebtoken = "^9.3.0"


# 用于对Trait提供异步支持
async-trait = "0.1.80"

# 用于base64编码
base64 = "0.22.1"

  
# web框架相关依赖  

[dependencies.rocket_dyn_templates]

version = "^0.2"

features = ["handlebars"]

  

[dependencies.rocket_contrib]

version = "^0.4"

features = ["json"]

  

[dependencies.diesel_migrations]

version = "^2.2"
```

> 截止至目前(24.5.26)依赖更新情况,以[crates.io](https://crates.io)为参考

| 依赖                                                                    | 版本       |
| --------------------------------------------------------------------- | -------- |
| [diesel](https://crates.io/crates/diesel)                             | v2.1.6   |
| [dotenvy](https://crates.io/crates/dotenvy)                           | v0.15.7  |
| [serde](https://crates.io/crates/serde)                               | v1.0.203 |
| [serde_json](https://crates.io/crates/serde_json)                     | v1.0.117 |
| [chrono](https://crates.io/crates/chrono)                             | v0.4.38  |
| [rocket](https://crates.io/crates/rocket)                             | v0.5.1   |
| [rocket_sync_db_pools](https://crates.io/crates/rocket_sync_db_pools) | v0.1.0   |
| [rand](https://crates.io/crates/rand)                                 | v0.8.5   |
| [libsqlite3-sys](https://crates.io/crates/libsqlite3-sys)             | v0.28.0  |
| [log](https://crates.io/crates/log)                                   | v0.4.21  |
| [env_logger](https://crates.io/crates/env_logger)                     | v0.11.3  |
| [rocket_dyn_templates](https://crates.io/crates/rocket_dyn_templates) | v0.2.0   |
| [rocket_contrib](https://crates.io/crates/rocket_contrib)             | v0.4.11  |
| [diesel_migrations](https://crates.io/crates/diesel_migrations)       | v2.2.0   |
*[依赖介绍](依赖)* 

## 在Rust项目中使用Diesel来管理这些表和数据操作

### 安装Diesel CLI
```sh
cargo install diesel_cli --no-default-features --features sqlite
```

### 初始化Diesel
```sh
diesel setup
diesel migration generate create_warehouse_schema
```

### Diesel数据库迁移文件

```up.SQL
-- 货品表
CREATE TABLE products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    category_id INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

-- 分类表
CREATE TABLE categories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT
);

-- 库存表
CREATE TABLE inventory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    product_id INTEGER NOT NULL,
    warehouse_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (warehouse_id) REFERENCES warehouses(id)
);

-- 仓库表
CREATE TABLE warehouses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    location TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 仓库调度表
CREATE TABLE warehouse_transfers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    product_id INTEGER NOT NULL,
    from_warehouse_id INTEGER NOT NULL,
    to_warehouse_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL,
    transfer_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (from_warehouse_id) REFERENCES warehouses(id),
    FOREIGN KEY (to_warehouse_id) REFERENCES warehouses(id)
);

-- 管理员表
CREATE TABLE administrators (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL,
    password TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

```down.sql
-- This file should undo anything in `up.sql`

DROP TABLE administrators;

DROP TABLE warehouse_transfers;

DROP TABLE warehouses;

DROP TABLE inventory;

DROP TABLE categories;

DROP TABLE products;
```

>此时可以手动运行迁移 `diesel migration run`.

### 后端实现

#### 后端重点

重点应放在建立完备的访问安全性基础设施和完备合理的错误处理上.


#### 网络访问的安全性

1. **P2P 通信加密和数字签名**：
   - **加密**：确保 P2P 通信使用强加密算法（如 AES、RSA）来保护数据的机密性。
   - **数字签名**：使用数字签名来验证消息的完整性和来源，确保数据未被篡改并且来自可信源。

2. **HTTP 接口和跨域资源访问**：
   - **CORS 配置**：在前端代理配置 CORS（跨域资源共享）规则时，确保只允许受信任的域名进行访问。
   - **安全头部**：配置 HTTP 安全头部，如 `Content-Security-Policy` 和 `Strict-Transport-Security`，进一步提升安全性。

3. **HTTPS 加密**：
   - **证书管理**：使用有效的 SSL/TLS 证书并定期更新，确保 HTTPS 加密的安全性。
   - **反向代理**：配置反向代理（如 Nginx、Apache）处理 HTTPS 加密，保护所有前端和后端通信。

4. **Cookie 安全性**：
   - **密钥加密**：使用强加密算法保护 cookie 数据。
   - **HttpOnly**：启用 `HttpOnly` 标志以防止客户端脚本访问 cookie，减少 XSS 攻击的风险。
   - **Secure**：设置 `Secure` 标志以确保 cookie 仅通过 HTTPS 传输。

#### 仓库管理系统的实际应用情况

1. **超级管理员行为限制**：
   - **权限管理**：实施细粒度权限控制，限制超级管理员的操作范围，防止误操作或恶意行为。
   - **记录标记**：使用标记（如软删除）而不是完全删除记录，确保历史记录的可追溯性和恢复能力。

2. **消息队列和日志记录**：
   - **消息队列**：使用可靠的消息队列系统（如 RabbitMQ、Kafka）来处理数据库读写请求，确保请求的顺序和完整性。
   - **日志记录**：实施日志记录策略，记录所有重要操作和事件，以便审计和问题排查。

3. **权限管理和访问控制**：
   -  **权限表和token**：使用权限表来制定权限，使用jwt实现网络接口的访问控制

#### 错误处理

1. **错误恢复和状态说明**：
   - **错误处理**：设计完善的错误处理机制，提供友好的错误信息和状态恢复选项，避免使用 `panic` 导致服务中断。
   - **用户提示**：为用户提供清晰的错误信息和建议，帮助他们理解问题并采取适当的行动。

2. **服务端日志记录**：
   - **日志策略**：制定详细的日志记录策略，包括记录级别（信息、警告、错误）和日志保存周期。
   - **监控和警报**：实施实时监控和警报系统，及时发现和响应服务异常或错误。

这些措施将有助于提升系统的安全性和稳定性。总体来看，你的设计已经涵盖了大部分关键领域，确保网络安全、数据安全以及错误处理的完整性。

#### 操作的原子性和可回滚

##### 多数据库的仓库管理系统使用分布式数据管理来协调多节点

**分布式事务管理**：
• **两阶段提交协议 (2PC)**：包括两个阶段：准备阶段和提交阶段。在准备阶段，协调者询问所有参与者是否可以提交事务；在提交阶段，协调者根据所有参与者的反馈决定是否提交或回滚事务。

##### 单一节点的数据操作的原子性和可回滚

**1. 事务管理**
- **SQLite 本身支持事务**，利用 SQLite 的事务机制来确保原子性。可以在多个数据库操作之间使用 BEGIN TRANSACTION 和 COMMIT / ROLLBACK 命令来确保操作的原子性。例如：
```Rust
let conn = Connection::open("database.db")?;
conn.execute("BEGIN TRANSACTION;", [])?;

// 执行多个数据库操作
conn.execute("INSERT INTO table1 (column1) VALUES (?);", [value1])?;
conn.execute("UPDATE table2 SET column2 = ? WHERE id = ?;", [value2, id])?;

// 如果所有操作成功
conn.execute("COMMIT;", [])?;
```

**2.对象关系映射（ORM）对事务管理的支持

- **ORM 框架提供了对事务的支持，包括**：
  - **开始事务**：初始化一个新的事务。
  - **提交事务**：将事务中的所有操作持久化到数据库。
  - **回滚事务**：撤销事务中的所有操作，恢复到事务开始之前的状态。

##### 文件锁机制
SQLite 使用文件锁来管理并发访问。当多个进程尝试同时访问同一个数据库文件时，SQLite 使用以下锁级别来确保数据的安全性和一致性：
- **SHARED**: 允许多个读取操作，但禁止写入操作。
- **RESERVED**: 写入操作已预订，但仍然允许其他读取操作。
- **PENDING**: 将进行写入操作，禁止新的读取操作。
- **EXCLUSIVE**: 完全锁定数据库，禁止任何其他读取或写入操作。
通过这种锁机制，SQLite 能够避免数据竞争条件，确保在多进程、多线程的环境下，数据操作的安全性。

##### SQLite 的数据库文件访问权限
SQLite 使用底层的文件系统权限来管理对数据库文件的访问。这些权限通常由操作系统的文件系统来管理，通过设置文件的权限来控制对数据库文件的访问。

**文件权限管理**
SQLite 数据库文件权限受操作系统的文件权限控制。在 Unix-Like 系统上（如 Linux、macOS），可以使用 chmod 来管理文件的访问权限。
- **r**: 读取权限
- **w**: 写入权限
- **x**: 执行权限（不适用于数据库文件）
通过这些权限，SQLite 可以控制哪些用户和进程可以读取、写入数据库文件。

##### 可回滚
- **在数据库表中记录每次提交的版本信息。
- **SQLite 的 WAL 模式文件**
当 SQLite 启用**WAL（Write-Ahead Logging）模式时，会生成两个额外的文件**：
- **.wal 文件**：用于记录写入操作，以便在事务提交前缓冲。
- **.shm 文件**：用于协调多个进程间的内存共享。
以各个节点的日志为辅助，可以复原历史修改

#### 定义对象关系映射结构体

```rust
// src/models.rs

  

use super::schema::*;

use serde::{Serialize, Deserialize};

use diesel::{Queryable, Identifiable, Insertable};

use diesel::prelude::*;

// use diesel::sql_types::Integer;

use chrono::NaiveDateTime;

  

#[derive(Queryable, Identifiable, Serialize, Deserialize, Associations)]

#[diesel(table_name = products)]

#[diesel(belongs_to(Category))]

pub struct Product {

    pub id: Option<i32>,

    pub name: String,

    pub description: Option<String>,

    pub category_id: Option<i32>,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}

  

#[derive(Queryable, Identifiable, Serialize, Deserialize, Insertable)]

#[diesel(table_name = categories)]

pub struct Category {

    pub id: Option<i32>,

    pub name: String,

    pub description: Option<String>,

}

  

#[derive(Selectable, Queryable, Identifiable, Serialize, Deserialize, Associations)]

#[diesel(table_name = inventory)]

#[diesel(belongs_to(Product))]

#[diesel(belongs_to(Warehouse))]

  

pub struct Inventory {

    pub id: Option<i32>,

    pub product_id: i32,

    pub warehouse_id: String,

    pub quantity: i32,

    pub deleted: Option<i32>,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}

  

#[derive(Queryable, Identifiable, Serialize, Deserialize, Insertable)]

  

#[diesel(table_name = warehouses)]

pub struct Warehouse {

    pub id: String,

    pub localkey: Option<String>,

    pub name: String,

    pub location: String,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}

  

#[derive(Queryable, Identifiable, Serialize, Deserialize, Associations)]

#[diesel(table_name = warehouse_transfers)]

#[diesel(belongs_to(Product))]

#[diesel(belongs_to(Warehouse, foreign_key = from_warehouse_id))]

pub struct WarehouseTransfer {

    pub id: i32,

    pub product_id: i32,

    pub from_warehouse_id: String,

    pub to_warehouse_id: String,

    pub quantity: i32,

    pub transfer_date: Option<NaiveDateTime>,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}

  

#[derive(Queryable, Identifiable, Serialize, Deserialize)]

#[diesel(table_name = administrators)]

pub struct Administrator {

    pub id: Option<i32>,

    pub username: String,

    pub password: String,

    pub superuser: bool,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}
```

#### 定义数据库模式

```rust
// src/schema.rs
// @generated automatically by Diesel CLI.

  
  
  

diesel::table! {

    administrators (id) {

        id -> Nullable<Integer>,

        username -> Text,

        password -> Text,

        superuser -> Bool,

        created_at -> Nullable<Timestamp>,

        updated_at -> Nullable<Timestamp>,

    }

}

  

diesel::table! {

    categories (id) {

        id -> Nullable<Integer>,

        name -> Text,

        description -> Nullable<Text>,

    }

}

  

diesel::table! {

    inventory (id) {

        id -> Nullable<Integer>,

        product_id -> Integer,

        warehouse_id -> Text,

        quantity -> Integer,

        deleted -> Nullable<Integer>,

        created_at -> Nullable<Timestamp>,

        updated_at -> Nullable<Timestamp>,

    }

}

  

diesel::table! {

    products (id) {

        id -> Nullable<Integer>,

        name -> Text,

        description -> Nullable<Text>,

        category_id -> Nullable<Integer>,

        created_at -> Nullable<Timestamp>,

        updated_at -> Nullable<Timestamp>,

    }

}

  

diesel::table! {

    warehouse_transfers (id) {

        id -> Integer,

        product_id -> Integer,

        from_warehouse_id -> Text,

        to_warehouse_id -> Text,

        quantity -> Integer,

        transfer_date -> Nullable<Timestamp>,

        created_at -> Nullable<Timestamp>,

        updated_at -> Nullable<Timestamp>,

    }

}

  

diesel::table! {

    warehouses (id) {

        id -> Text,

        localkey -> Nullable<Text>,

        name -> Text,

        location -> Text,

        created_at -> Nullable<Timestamp>,

        updated_at -> Nullable<Timestamp>,

    }

}

  

diesel::joinable!(inventory -> products (product_id));

diesel::joinable!(inventory -> warehouses (warehouse_id));

diesel::joinable!(products -> categories (category_id));

diesel::joinable!(warehouse_transfers -> products (product_id));

  

diesel::allow_tables_to_appear_in_same_query!(

    administrators,

    categories,

    inventory,

    products,

    warehouse_transfers,

    warehouses,

);
```

#### 引用模块和插件

```rust
//src/main.rs

#[macro_use]

extern crate rocket;


extern crate base64;

  

// use rocket::fairing::AdHoc;

use rocket::fairing::{Fairing, Info, Kind};

use rocket::figment::{

    providers::{Env, Serialized},

    Figment,

};

use rocket::http::Status;

use rocket::request::{FromRequest, Outcome};

// use rocket::request;

// use rocket::response::status;

use rocket::http::Method;

use rocket::http::{Cookie, CookieJar};

// use rocket::response::status::Custom;

use rocket::serde::json::Json;

use rocket::Request;

use rocket::{Build, Config, Rocket};

use rocket_cors::{AllowedHeaders, AllowedOrigins, CorsOptions};

use rocket_sync_db_pools::{database, diesel};

  

use crate::models::*;

use crate::schema::*;

use diesel::dsl::sql;

use diesel::prelude::*;

use diesel::sql_types::Integer;

use diesel::sql_types::Nullable;

use diesel::sqlite::SqliteConnection;

use diesel_migrations::{embed_migrations, EmbeddedMigrations, MigrationHarness};

use rand::distributions::Alphanumeric;

use rand::Rng;

use std::collections::HashMap;

use std::collections::HashSet;

use std::error::Error;

// use std::io::Cursor;

use std::result::Result as StdResult; // 为了避免名称冲突，使用别名

                                      // use std::time::Duration;

use std::env;

use std::time::{SystemTime, UNIX_EPOCH};

// use std::clone;

  

use chrono::Local;

use chrono::NaiveDateTime;

use chrono::Utc;

use serde::Deserialize;

use serde::Serialize;

// use uuid::Uuid;

  

use libp2p::floodsub::{Floodsub, FloodsubEvent, Topic};

// use libp2p::floodsub;

use libp2p::futures::StreamExt;

use libp2p::identity::{self, ed25519, PublicKey};

// use libp2p::identity::Keypair;

use libp2p::kad::{record::store::MemoryStore, Kademlia, KademliaConfig, KademliaEvent};

  

use libp2p::mdns::{Mdns, MdnsConfig, MdnsEvent};

use libp2p::ping::{Ping, PingConfig, PingEvent};

// use libp2p::request_response::{

//     ProtocolName, RequestResponse, RequestResponseCodec, RequestResponseEvent,

//     RequestResponseMessage,

// };

use libp2p::swarm::{Swarm, SwarmBuilder, SwarmEvent};

  

// use libp2p::swarm::{

//     NetworkBehaviour, NetworkBehaviourEventProcess

// };

use libp2p::{development_transport, Multiaddr, NetworkBehaviour, PeerId};

  

use tokio;

// use tokio::io::{ AsyncBufReadExt, AsyncRead, AsyncReadExt, AsyncWrite, AsyncWriteExt};

use tokio::io;

use tokio::signal;

use tokio::task;

use tokio_util::compat::TokioAsyncReadCompatExt;

// use tokio_util::compat::FuturesAsyncReadCompatExt;

// use libp2p::dns::DnsConfig;

// use libp2p::tcp::GenTcpConfig;

  

use log::LevelFilter;

use log::{error, info, warn};

// use log::debug;

use log4rs::append::console::ConsoleAppender;

use log4rs::append::file::FileAppender;

use log4rs::config::Config as LogConfig;

use log4rs::config::{Appender, Root};

use log4rs::encode::pattern::PatternEncoder;

use log4rs::init_config;

  

use base64::{engine::general_purpose, Engine as _};

use futures::prelude::*;

  

use jsonwebtoken::{decode, encode, DecodingKey, EncodingKey, Header, Validation};

  

mod models;

mod schema;
```


#### 实现Trait

```
impl Claims {

    fn new(username: &str) -> Self {

        let now = SystemTime::now();

        let exp = now

            .duration_since(UNIX_EPOCH)

            .expect("Time went backwards")

            .as_secs()

            + 1800; // Token expires in 30 minutes

  

        Self {

            sub: username.to_owned(),

            exp: exp as usize,

        }

    }

}//cookieToken负载的实现,包含用户名和过期时间


impl From<KademliaEvent> for KMBehaviourEvent {

    fn from(event: KademliaEvent) -> Self {

        KMBehaviourEvent::Kademlia(event)

    }

}//基于Kademlia协议的广域网网络发现的实现

  

impl From<MdnsEvent> for KMBehaviourEvent {

    fn from(event: MdnsEvent) -> Self {

        KMBehaviourEvent::Mdns(event)

    }

}//基于mDNS协议的局域网网络发现的实现

  

impl From<PingEvent> for KMBehaviourEvent {

    fn from(event: PingEvent) -> Self {

        KMBehaviourEvent::Ping(event)

    }

}//p2p网络节点之间ping行为的实现

  

impl From<FloodsubEvent> for KMBehaviourEvent {

    fn from(event: FloodsubEvent) -> Self {

        KMBehaviourEvent::Floodsub(event)

    }

}//p2p网络节点之间消息发布和订阅行为的实现

  

#[async_trait]

impl<'r> FromRequest<'r> for JwtToken {

    type Error = ();

  

    async fn from_request(request: &'r Request<'_>) -> rocket::request::Outcome<Self, Self::Error> {

        let cookies: &CookieJar<'_> = request.cookies();

        if let Some(cookie) = cookies.get("token") {

            let token = cookie.value();

            match decode_token(token) {

                Some(username) => Outcome::Success(JwtToken(username)),

                None => Outcome::Error((Status::Unauthorized, ())),

            }

        } else {

            Outcome::Error((Status::Unauthorized, ()))

        }

    }

}//rocket获取cookie和token的实现
```

> cookies获取必须是同域请求

#### 定义行为等结构体

```rust
#[derive(Debug)]

struct AdminInit;

#[derive(Debug)]

pub struct JwtToken(pub String);

  

#[derive(Debug, Serialize, Deserialize)]

struct Claims {

    sub: String,

    exp: usize,

}

#[derive(Debug, Serialize, Deserialize)]

pub struct CustomResponder {

    status: Status,

    message: String,

}

  

#[derive(FromForm, Deserialize)]

struct LoginCredentials {

    username: String,

    password: String,

}

#[derive(NetworkBehaviour)]

#[behaviour(out_event = "KMBehaviourEvent")]

struct KMBehaviour {

    kademlia: Kademlia<MemoryStore>,

    mdns: Mdns,

    ping: Ping,

    floodsub: Floodsub,

}

  

enum KMBehaviourEvent {

    Kademlia(KademliaEvent),

    Mdns(MdnsEvent),

    Ping(PingEvent),

    Floodsub(FloodsubEvent),

}

#[database("sqlite_db")]

pub struct DbConn(SqliteConnection);

  

#[get("/")]

fn index() -> &'static str {

    "Hello, world!"

}

  

#[derive(Insertable, Deserialize)]

#[diesel(table_name = warehouses)]

pub struct NewWarehouse {

    pub id: String,

    pub name: String,

    pub location: String,

}

  

#[derive(Insertable, Serialize, Deserialize)]

#[diesel(table_name = categories)]

pub struct NewCategory {

    pub name: String,

    pub description: Option<String>,

}

  

#[derive(Insertable, Serialize, Deserialize)]

#[diesel(table_name = products)]

pub struct NewProduct {

    pub name: String,

    pub description: Option<String>,

    pub category_id: Option<i32>,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}

  

#[derive(Insertable, Serialize, Deserialize)]

#[diesel(table_name = administrators)]

pub struct NewAdministrator {

    pub username: String,

    pub password: String,

    pub superuser: bool,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}

#[derive(Queryable, Serialize, Deserialize)]

#[diesel(table_name = warehouses)]

pub struct GetWarehouses {

    pub id: String,

    pub name: String,

    pub location: String,

    pub created_at: Option<NaiveDateTime>,

    pub updated_at: Option<NaiveDateTime>,

}
```

#### 数据库连接函数

调用后返回SqliteConnection结构,如果目标位置的嵌入式数据库文件不存在,则新建一个数据库.过程自动完成并且生成数据库所需的系统库嵌入到构建中.数据库连接函数只会在程序启动和初始化时使用,启动或初始化行为结束后即释放

```
// 建立数据库连接

fn establish_connection() -> SqliteConnection {

    let database_url = "sqlite://./warehouse.db";

    SqliteConnection::establish(&database_url)

        .expect(&format!("Error connecting to {}", database_url))

}
```


#### 一些初始化函数

##### 数据迁移函数
```
async fn run_db_migrations(conn: &mut SqliteConnection) {

    const MIGRATIONS: EmbeddedMigrations = embed_migrations!();

    if let Err(err) = conn.run_pending_migrations(MIGRATIONS) {

        error!("Error running migrations: {}", err);

    } else {

        info!("Database migrations executed successfully");

    }

}
```

通过调用此函数,在应用启动时自动依据嵌入的数据库迁移文件来运行数据迁移

##### 本地节点初始化函数

```
// 获取名为"ThisWarehouse"的仓库ID MpHCXo8e0RfSru1kQQKoJawUgEUs9oYmktHPF+bT26o=

fn get_warehouse_id(

    conn: &mut SqliteConnection,

) -> Result<ed25519::Keypair, diesel::result::Error> {

    use self::warehouses::dsl::*;

    let warehouse: Warehouse = warehouses.filter(localkey.is_not_null()).first(conn)?;

    info!("Found warehouse: {:?}", warehouse.localkey);

    let mut local_key_bytes = if let Some(local_key) = &warehouse.localkey {

        general_purpose::STANDARD

            .decode(local_key)

            .expect("Base64 decode error")

    } else {

        Vec::new() // 或者其他默认值的处理

    };

    let keypair =

        ed25519::Keypair::decode(local_key_bytes.as_mut_slice()).expect("Keypair decode error");

    Ok(keypair)

}

  

fn generate_and_insert_new_local_key(conn: &mut SqliteConnection) -> ed25519::Keypair {

    let local_key = ed25519::Keypair::generate();

    let local_key_base64 = general_purpose::STANDARD.encode(local_key.encode());

    info!(

        "Generated and inserted new key {:?} for warehouse ThisWarehouse",

        local_key_base64

    );

    let local_peer_id = PeerId::from(PublicKey::Ed25519(local_key.public()));

    let new_warehouse = Warehouse {

        id: local_peer_id.to_string(),

        localkey: Some(local_key_base64),

        name: "ThisWarehouse".to_string(),

        location: "/ip4/127.0.0.1/tcp/8080".to_string(),

        created_at: Some(Utc::now().naive_utc()),

        updated_at: Some(Utc::now().naive_utc()),

    };

  

    use self::warehouses::dsl::*;

    diesel::insert_into(warehouses)

        .values(&new_warehouse)

        .execute(conn)

        .expect("Error inserting new warehouse");

  

    local_key

}
```

这里查找了数据库中本地节点的相关记录.
如果没有找到则会运行初始化

初始化过程包括生成ed25519密钥对,通过base64编码存入数据库本地节点记录中,从环境变量读取本地节点所在网络位置,存入数据库本地节点记录

程序启动时会读取密钥,依据公钥生成供其他节点认证的PeerID并打印到控制台,缓存私钥以供认证.

##### 本地节点superuser初始化的行为

```rust
#[rocket::async_trait]

impl Fairing for AdminInit {

    fn info(&self) -> Info {

        Info {

            name: "Admin Initialization",

            kind: Kind::Ignite,

        }

    }

  

    async fn on_ignite(&self, rocket: Rocket<Build>) -> Result<Rocket<Build>, Rocket<Build>> {

        let db = DbConn::get_one(&rocket).await.expect("database connection");

  

        db.run(|c| {

            use self::administrators::dsl::*;

  

            let admin_count: i64 = administrators

                .count()

                .get_result(c)

                .expect("Error counting admins");

  

            if admin_count == 0 {

                let mut random_password = String::from("111");

  

                #[cfg(not(debug_assertions))]

                {

                    random_password = rand::thread_rng()

                        .sample_iter(&Alphanumeric)

                        .take(12)

                        .map(char::from)

                        .collect();

                }

  

                info!("默认管理员已创建，用户名: admin, 密码: {}", random_password);

                diesel::insert_into(administrators)

                    .values((

                        username.eq("admin"),

                        password.eq(random_password),

                        superuser.eq(true),

                    ))

                    .execute(c)

                    .expect("Error inserting admin");

            }

        })

        .await;

  

        Ok(rocket)

    }

}
```

首次启动rocket会通过此行为生成默认管理员,身份为superuser,并把登录信息写入启动日志来供用户连接.在debug构建中密码为111,在非debug构建中会使用随机密码.

#### 路由功能的实现

##### 登录和注册的options路由

```rust
#[rocket::options("/login")]

fn options_login() -> Status {

    Status::Ok // 返回一个允许的状态码，如 200 OK

}

#[rocket::options("/signup")]

fn options_signup() -> Status {

    Status::Ok // 返回一个允许的状态码，如 200 OK

}
```

##### 测试用保护路由

```rust
// 受保护的路由

#[get("/protected")]

fn protected_route(token: JwtToken) -> String {

    format!("Hello, {}! This is a protected route.", token.0)

}
```

##### 各种功能路由

```rust
#[get("/warehouses")]

async fn get_warehouses(

    conn: DbConn,

    token: JwtToken

) -> Result<Json<Vec<GetWarehouses>>, Json<CustomResponder>> {

    use schema::administrators::dsl::*;

    conn.run(move |c| {

        let admin = administrators

            .filter(username.eq(token.0.clone()))

            .first::<Administrator>(c)

            .optional()

            .map_err(|_| Json(CustomResponder{ status:rocket::http::Status::InternalServerError, message: "Error checking for administrator".to_string()}))?;

        if admin.is_none() {

            return Err(Json(CustomResponder{ status:rocket::http::Status::Unauthorized, message: "Permission Denied".to_string()}));

        }

        use schema::warehouses::dsl::*;

  

        let result = warehouses

            .select((id, name, location, created_at, updated_at)) // 选择除去localkey之外的列

            .load::<GetWarehouses>(c)

            .map(Json)

            .expect("Error loading warehouses");

        Ok(result)

    })

    .await

}

  

#[post("/products", format = "application/json", data = "<new_product>")]

async fn create_product(

    conn: DbConn,

    new_product: Json<NewProduct>,

    token: JwtToken,

) -> Result<Json<NewProduct>, Json<CustomResponder>> {

    let db = conn;

    use schema::administrators::dsl::*;

    let new_product = db

        .run(move |c| {

            let admin = administrators

                .filter(username.eq(token.0.clone()))

                .first::<Administrator>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for administrator".to_string(),

                    })

                })?;

  

            if admin.is_none() {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::Unauthorized,

                    message: "Permission Denied".to_string(),

                }));

            }

  

            use crate::schema::products::dsl::*;

            let new_product = NewProduct {

                name: new_product.name.clone(),

                description: new_product.description.clone(),

                category_id: new_product.category_id.clone(),

                created_at: Some(Utc::now().naive_utc()),

                updated_at: Some(Utc::now().naive_utc()),

            };

  

            let existing_product = products

                .filter(name.eq(&new_product.name))

                .first::<Product>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for product".to_string(),

                    })

                })?;

  

            if let Some(_) = existing_product {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::BadRequest,

                    message: "Product with this name already exists".to_string(),

                }));

            }

  

            diesel::insert_into(products)

                .values(&new_product)

                .execute(c)

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error inserting new product".to_string(),

                    })

                })?;

            Ok(Json(new_product))

        })

        .await?;

    Ok(new_product)

}

  

#[post("/categories", format = "application/json", data = "<new_category>")]

async fn create_category(

    conn: DbConn,

    new_category: Json<NewCategory>,

    token: JwtToken,

) -> Result<Json<NewCategory>, Json<CustomResponder>> {

    let db = conn;

    use schema::administrators::dsl::*;

    let new_category = db

        .run(move |c| {

            let admin = administrators

                .filter(username.eq(token.0.clone()))

                .first::<Administrator>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for administrator".to_string(),

                    })

                })?;

  

            if admin.is_none() {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::Unauthorized,

                    message: "Permission Denied".to_string(),

                }));

            }

            use crate::schema::categories::dsl::*;

            let new_category = NewCategory {

                name: new_category.name.clone(),

                description: new_category.description.clone(),

            };

  

            let existing_category = categories

                .filter(name.eq(&new_category.name))

                .first::<Category>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for category".to_string(),

                    })

                })?;

  

            if let Some(_) = existing_category {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::BadRequest,

                    message: "Category with this name already exists".to_string(),

                }));

            }

  

            diesel::insert_into(categories)

                .values(&new_category)

                .execute(c)

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error inserting new category".to_string(),

                    })

                })?;

            Ok(Json(new_category))

        })

        .await?;

    Ok(new_category)

}

  

#[post("/warehouses", format = "json", data = "<new_warehouse>")]

async fn create_warehouse(

    new_warehouse: Json<NewWarehouse>,

    conn: DbConn,

    token: JwtToken,

) -> Result<Json<Warehouse>, Json<CustomResponder>> {

    let db = conn;

    use schema::administrators::dsl::*;

    let new_warehouse = db

        .run(move |c| {

            let admin = administrators

                .filter(username.eq(token.0.clone()))

                .first::<Administrator>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for administrator".to_string(),

                    })

                })?;

  

            if admin.is_none() {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::Unauthorized,

                    message: "Permission Denied".to_string(),

                }));

            }

  

            use self::schema::warehouses::dsl::*;

  

            let existing_warehouse = warehouses

                .filter(location.eq(&new_warehouse.location))

                .first::<Warehouse>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for warehouses".to_string(),

                    })

                })?;

  

            if let Some(_) = existing_warehouse {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::BadRequest,

                    message: "Warehouse with this location already exists".to_string(),

                }));

            }

  

            let new_warehouse = Warehouse {

                id: new_warehouse.id.clone(),

                localkey: Some("".to_string()),

                name: new_warehouse.name.clone(),

                location: new_warehouse.location.clone(),

                created_at: Some(Utc::now().naive_utc()),

                updated_at: Some(Utc::now().naive_utc()),

            };

  

            diesel::insert_into(warehouses)

                .values(&new_warehouse)

                .execute(c)

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error inserting new warehouse".to_string(),

                    })

                })?;

  

            Ok(Json(new_warehouse))

        })

        .await?;

    Ok(new_warehouse)

}

  

#[get("/products?<start>&<end>&<categories_name>")]

async fn get_products(

    conn: DbConn,

    start: String,

    end: String,

    categories_name: Option<String>,

    token: JwtToken,

) -> Result<Json<Vec<Product>>, Json<CustomResponder>> {

    use chrono::NaiveDateTime;

    use diesel::prelude::*;

    use schema::administrators::dsl::*;

    use schema::categories::dsl::{categories, id as cat_id, name as cat_name};

    use schema::products::dsl::{category_id, created_at, products};

  

    // 解析start和end为NaiveDateTime

    let start_dt = NaiveDateTime::parse_from_str(&start, "%Y-%m-%d %H:%M:%S").unwrap();

    let end_dt = NaiveDateTime::parse_from_str(&end, "%Y-%m-%d %H:%M:%S").unwrap();

  

    let results = conn

        .run(move |c| {

            let admin = administrators

                .filter(username.eq(token.0.clone()))

                .first::<Administrator>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for administrator".to_string(),

                    })

                })?;

  

            if admin.is_none() {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::Unauthorized,

                    message: "Permission Denied".to_string(),

                }));

            }

  

            let mut query = products

                .filter(created_at.between(start_dt, end_dt))

                .into_boxed();

  

            // 如果categories_name有值，添加分类名过滤条件

            if let Some(cat_name_filter) = categories_name {

                query = query.filter(

                    category_id.eq_any(

                        categories

                            .filter(cat_name.eq(cat_name_filter))

                            .select(cat_id),

                    ),

                );

            }

  

            Ok(query.load::<Product>(c).expect("Error loading warehouses")) // 如果加载或转换过程中出现错误，抛出一个错误

        })

        .await;

  

    results.map(Json)

}

  

#[get("/inventory?<start>&<end>&<product_name>")]

async fn get_inventory(

    conn: DbConn,

    start: String,

    end: String,

    product_name: Option<String>,

    token: JwtToken,

) -> Result<Json<Vec<Inventory>>, Json<CustomResponder>> {

    use chrono::NaiveDateTime;

    use diesel::prelude::*;

    use schema::administrators::dsl::*;

  

    use schema::inventory::dsl::{created_at,inventory, product_id};

    use schema::products::dsl::{name as pro_name, products};

  

    // 解析start和end为NaiveDateTime

    let start_dt = NaiveDateTime::parse_from_str(&start, "%Y-%m-%d %H:%M:%S").unwrap();

    let end_dt = NaiveDateTime::parse_from_str(&end, "%Y-%m-%d %H:%M:%S").unwrap();

  

    let results = conn

        .run(move |c| {

            let admin = administrators

                .filter(username.eq(token.0.clone()))

                .first::<Administrator>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for administrator".to_string(),

                    })

                })?;

  

            if admin.is_none() {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::Unauthorized,

                    message: "Permission Denied".to_string(),

                }));

            }

  

            let mut query = inventory

                .filter(created_at.between(start_dt, end_dt))

                .into_boxed();

  

            if let Some(product_name_filter) = &product_name {

                // 直接将 pro_id 的类型声明为 Integer 而不是 Nullable<Integer>

                let subquery = products

                    .filter(pro_name.eq(product_name_filter))

                    .select(sql::<Integer>("COALESCE(pro_id, 0)"));

                query = query.filter(product_id.eq_any(subquery));

            }

  

            Ok(query

                .load::<Inventory>(c)

                .expect("Error loading warehouses")) // 如果加载或转换过程中出现错误，抛出一个错误

        })

        .await;

  

    results.map(Json)

}

  

// 登录接口

#[post("/login", data = "<credentials>")]

async fn login(

    credentials: Json<LoginCredentials>,

    conn: DbConn,

    cookies: &CookieJar<'_>,

) -> Result<Json<CustomResponder>, Json<CustomResponder>> {

    use schema::administrators::dsl::*;

  

    let input_username = credentials.username.clone(); // 提取用户名

    let input_password = credentials.password.clone(); // 提取密码，并用不同的变量名

  

    let result = conn

        .run(move |c| {

            administrators

                .filter(username.eq(&input_username))

                .first::<Administrator>(c)

                .optional()

        })

        .await;

  

    match result {

        Ok(Some(administrator)) if administrator.password == input_password => {

            let token = generate_token(&administrator.username);

            // 将 token 存入 Cookie 中

            let cookie: Cookie = Cookie::build(("token", token.clone()))

                .path("/")

                .secure(false)

                .http_only(true)

                .build();

            cookies.add(cookie);

            Ok(Json(CustomResponder {

                status: rocket::http::Status::Ok,

                message: "Success".to_string(),

            }))

        }

        _ => Err(Json(CustomResponder {

            status: rocket::http::Status::InternalServerError,

            message: "Error checking for administrator".to_string(),

        })),

    }

}

  

#[post("/signup", data = "<new_administrator>")]

async fn signup(

    new_administrator: Json<NewAdministrator>,

    conn: DbConn,

    token: JwtToken,

) -> Result<Json<CustomResponder>, Json<CustomResponder>> {

    use schema::administrators::dsl::*;

    let input_username = new_administrator.username.clone(); // 提取用户名

    let input_password = new_administrator.password.clone(); // 提取密码，并用不同的变量名

    let input_superuser = new_administrator.superuser.clone();

    let result = conn

        .run(move |c| {

            let admin = administrators

                .filter(username.eq(token.0))

                .filter(superuser.eq(true))

                .first::<Administrator>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::InternalServerError,

                        message: "Error checking for administrator".to_string(),

                    })

                })?;

            if admin.is_none() {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::Unauthorized,

                    message: "Permission Denied".to_string(),

                }));

            }

            let existing_admin = administrators

                .filter(username.eq(input_username.clone()))

                .first::<Administrator>(c)

                .optional()

                .map_err(|_| {

                    Json(CustomResponder {

                        status: rocket::http::Status::Unauthorized,

                        message: "Error checking for administrator".to_string(),

                    })

                })?;

            if let Some(_) = existing_admin {

                return Err(Json(CustomResponder {

                    status: rocket::http::Status::BadRequest,

                    message: "Administrator with this username already exists".to_string(),

                }));

            }

            let new_administrator = NewAdministrator {

                username: input_username,

                password: input_password,

                superuser: input_superuser,

                created_at: Some(Utc::now().naive_utc()),

                updated_at: Some(Utc::now().naive_utc()),

            };

            diesel::insert_into(administrators)

                .values(&new_administrator)

                .execute(c)

                .expect("Error inserting admin");

            Ok(Json(CustomResponder {

                status: rocket::http::Status::Ok,

                message: "Administrator created".to_string(),

            }))

        })

        .await;

    result

}
```

路由中的数据库行为并不是通过上面的数据库连接函数实现的,而是rocket挂载的数据连接池.
路由的返回值为`Json<CustomResponder>`这一定制的结构体,包含了http请求状态和回应信息

#### 构建rocket

```rust
async fn rocket() -> Rocket<Build> {

    // 从默认配置创建 Figment 实例

    let figment = Figment::from(Config::default())

        .merge(("port", 0))

        // 合并自定义的数据库配置

        .merge(Serialized::default("databases", {

            let mut databases: HashMap<&str, HashMap<&str, &str>> = HashMap::new();

            let mut db_config: HashMap<&str, &str> = HashMap::new();

            db_config.insert("url", "sqlite://./warehouse.db");

            databases.insert("sqlite_db", db_config);

            databases

        }))

        // 合并环境变量配置，前缀为 "APP_"

        .merge(Env::prefixed("APP_"));

  

    // 使用自定义的配置启动 Rocket 应用程序

    let mut rocket = rocket::custom(figment)

        // 附加数据库连接

        .attach(DbConn::fairing())
		//执行用于网页认证的管理员初始化
        .attach(AdminInit)

        // 挂载路由

        .mount("/", routes![index])

        .mount(

            "/api",

            routes![

                get_warehouses,

                create_warehouse,

                create_category,

                create_product,

                get_products,

                login,

                options_login,

                protected_route,

                signup,

                options_signup

            ],

        );//挂载路由

  

    // 只在 debug 模式下启用 CORS 配置

    #[cfg(debug_assertions)]

    {

        let cors = CorsOptions::default()

            .allowed_origins(AllowedOrigins::all())

            .allowed_methods(

                vec![

                    Method::Get,

                    Method::Post,

                    // 还可以添加其它允许的方法，如 Method::Put, Method::Delete 等

                ]

                .into_iter()

                .map(From::from)

                .collect::<HashSet<_>>(),

            )

            .allowed_headers(AllowedHeaders::all())

            .allow_credentials(true)

            .to_cors()

            .unwrap();

        rocket = rocket.attach(cors);

    }

  

    rocket

}
```

> 在debug构建中使用跨域资源共享来方便调试,在release构建中关闭来加强安全性.
> 使用figment嵌入式配置来配置连接池,端口等配置

#### main函数

```
#[tokio::main]

async fn main() -> StdResult<(), Box<dyn Error>> {

    // 获取当前时间并格式化为文件名

    let now = Local::now();

    let log_file_name = format!("log/output_{}.log", now.format("%Y-%m-%d_%H-%M-%S"));

  

    // 构建文件 appender

    let file_appender = FileAppender::builder()

        .encoder(Box::new(PatternEncoder::new("{d} - {l} - {m}{n}")))

        .build(log_file_name)

        .unwrap();

  

    // 构建 console appender

    let console_appender = ConsoleAppender::builder()

        .encoder(Box::new(PatternEncoder::new("{d} - {l} - {m}{n}")))

        .build();

  

    // 创建 root 配置

    let logconfig = LogConfig::builder()

        .appender(Appender::builder().build("stdout", Box::new(console_appender)))

        .appender(Appender::builder().build("file", Box::new(file_appender)))

        .build(

            Root::builder()

                .appender("stdout")

                .appender("file")

                .build(LevelFilter::Info),

        )

        .unwrap();

  

    // 初始化 log4rs 配置

    init_config(logconfig).unwrap();

  

    // 创建本地PeerId

    let mut connection = establish_connection();

    run_db_migrations(&mut connection).await;

    let local_key = match get_warehouse_id(&mut connection) {

        Ok(id) => id,

        Err(err) => {

            error!("Failed to get warehouse local key: {}", err);

            generate_and_insert_new_local_key(&mut connection)

        }

    };

  

    let local_peer_id = PeerId::from(PublicKey::Ed25519(local_key.public()));

    info!("Local peer id: {:?}", local_peer_id);

    let floodsub = Floodsub::new(local_peer_id.clone());

    // 创建传输层

    let transport = development_transport(libp2p::identity::Keypair::Ed25519(local_key)).await?;

  

    // 创建Kademlia DHT

    let store = MemoryStore::new(local_peer_id.clone());

    let kademlia_config = KademliaConfig::default();

    let kademlia = Kademlia::with_config(local_peer_id.clone(), store, kademlia_config);

  

    // 创建mDNS

    let mdns = Mdns::new(MdnsConfig::default()).await?;

  

    let ping = Ping::new(PingConfig::new().with_keep_alive(true));

  

    // 创建网络行为

    let behaviour = KMBehaviour {

        kademlia,

        mdns,

        ping,

        floodsub,

    };

  

    // 构建Swarm

    let mut swarm = SwarmBuilder::new(transport, behaviour, local_peer_id)

        .executor(Box::new(|fut| {

            tokio::spawn(fut);

        }))

        .build();

  

    let topic = Topic::new("dev");

    // 获取环境变量中的 bootstrap_peer_id

    let bootstrap_peer_id_str = match env::var("BOOTSTRAP_PEER_ID") {

        Ok(val) => {

            info!("Find BOOTSTRAP_PEER_ID {:?}", val);

            val

        }

        Err(_) => {

            warn!("Warning: BOOTSTRAP_PEER_ID environment variable is not set");

            PeerId::from_public_key(&identity::PublicKey::Ed25519(

                identity::ed25519::PublicKey::decode(&[0u8; 32]).unwrap(),

            ))

            .to_string()

        }

    };

  

    // 获取环境变量中的 bootstrap_addr

    let bootstrap_addr_str = match env::var("BOOTSTRAP_ADDR") {

        Ok(val) => {

            info!("Find BOOTSTRAP_ADDR {:?}", val);

            val

        }

        Err(_) => {

            warn!("Warning: BOOTSTRAP_ADDR environment variable is not set");

            "/ip4/127.0.0.1/tcp/8080".to_string()

        }

    };

    // 获取环境变量中的 listening_addr_str

    let listening_addr_str = match env::var("LISTENING_ADDR") {

        Ok(val) => {

            info!("Find LISTENING_ADDR {:?}", val);

            val

        }

        Err(_) => {

            warn!("Warning: LISTENING_ADDR environment variable is not set, using default value:/ip4/0.0.0.0/tcp/12345");

            "/ip4/0.0.0.0/tcp/12345".to_string()

        }

    };

    // 添加引导节点

    let bootstrap_peer_id = bootstrap_peer_id_str.parse::<PeerId>()?;

    let bootstrap_addr: Multiaddr = bootstrap_addr_str.parse().unwrap();

  

    match env::var_os("BOOTSTRAP_ADDR") {

        Some(addr_value) => {

            info!("find BOOTSTRAP_ADDR {:?}", addr_value);

            match env::var_os("BOOTSTRAP_PEER_ID") {

                Some(id_value) => {

                    info!(

                        "via {:?} discover node which peer id is {:?}",

                        addr_value, id_value

                    );

                    swarm

                        .behaviour_mut()

                        .kademlia

                        .add_address(&bootstrap_peer_id, bootstrap_addr);

                }

                None => {

                    warn!("BOOTSTRAP_PEER_ID environment variable is not set,run as bootstrap node")

                }

            }

        }

        None => warn!("BOOTSTRAP_ADDR environment variable is not set,run as bootstrap node"),

    }

    // 设置监听地址

    let listen_addr: Multiaddr = env::var("LISTEN_ADDR")

        .unwrap_or_else(|_| listening_addr_str.to_string())

        .parse()

        .unwrap();

    Swarm::behaviour_mut(&mut swarm)

        .floodsub

        .subscribe(topic.clone());

  

    Swarm::listen_on(&mut swarm, listen_addr)?;

  

    let rocket_handle = task::spawn(async {

        let rocket = rocket().await;

        rocket.launch().await.unwrap();

    });

  

    let swarm_handle = task::spawn(async move {

        let mut discovered_peers = HashSet::new();

        let mut stdin = io::BufReader::new(io::stdin()).compat().lines();

        loop {

            tokio::select! {

                    line = stdin.next() => {

                        match line {

                            Some(Ok(line)) => {

                                let _ = swarm.behaviour_mut().floodsub.publish(topic.clone(), line.as_bytes());

                                info!("floodsub {:?} publish: {:?}", topic.clone(), line);

                            }

                            Some(Err(e)) => {

                                info!("Error reading from stdin: {:?}", e);

                            }

                            None => break, // EOF reached

                        }

                    }

                    event = swarm.next() => {

                        match event {

                            Some(event) => match event {

                                SwarmEvent::Behaviour(KMBehaviourEvent::Mdns(MdnsEvent::Discovered(peers))) => {

                                    for (peer_id, _) in peers {

                                        if discovered_peers.insert(peer_id.clone()) {

                                            info!("Discovered peer via mDNS: {:?}", peer_id);

                                            if let Err(e) = swarm.dial(peer_id.clone()) {

                                                info!("Failed to dial discovered peer: {:?}", e);

                                            }

                                        }

                                    }

                                }

                                SwarmEvent::Behaviour(KMBehaviourEvent::Mdns(MdnsEvent::Expired(peers))) => {

                                    for (peer_id, _) in peers {

                                        info!("Expired peer via mDNS: {:?}", peer_id);

                                    }

                                }

                                SwarmEvent::Behaviour(KMBehaviourEvent::Kademlia(

                                    KademliaEvent::RoutingUpdated { peer, .. },

                                )) => {

                                    if discovered_peers.insert(peer.clone()) {

                                        info!("Discovered peer via Kademlia: {:?}", peer);

                                        if let Err(e) = swarm.dial(peer) {

                                            info!("Failed to dial discovered peer: {:?}", e);

                                        }

                                    }

                                }

                                SwarmEvent::ConnectionEstablished { peer_id, .. } => {

                                    info!("Connected to peer: {:?}", peer_id);

                                }

                                SwarmEvent::ConnectionClosed { peer_id, cause, .. } => {

                                    if let Some(err) = cause {

                                        info!(

                                            "Connection to peer {:?} closed with error: {:?}",

                                            peer_id, err

                                        );

                                    } else {

                                        info!("Connection to peer {:?} closed.", peer_id);

                                    }

                                }

                                SwarmEvent::Behaviour(KMBehaviourEvent::Ping(event)) => {

                                    info!("Ping: {:?}", event);

                                }

                                SwarmEvent::Behaviour(KMBehaviourEvent::Floodsub(FloodsubEvent::Message(message))) => {

                                    info!("Received: '{:?}' from {:?}", String::from_utf8_lossy(&message.data), message.source);

                                }

                                _ => { }

                            },

                            None => break, // Swarm event stream ended

                        }

                    }

            }

        }

    });

  

    let mut rocket_handle = Some(rocket_handle);

    let mut swarm_handle = Some(swarm_handle);

  

    // 处理 SIGINT 信号

    tokio::select! {

        _ = signal::ctrl_c() => {

            warn!("Received shutdown signal, shutting down...");

        }

        _ = async { if let Some(handle) = rocket_handle.take() { handle.await.unwrap(); } } => {

            info!("Rocket task completed.");

        }

        _ = async { if let Some(handle) = swarm_handle.take() { handle.await.unwrap(); } } => {

            info!("Swarm task completed.");

        }

    }

  

    info!("Waiting for tasks to finish...");

    if let Some(handle) = rocket_handle {

        handle.await?;

    }

    if let Some(handle) = swarm_handle {

        handle.await?;

    }

  

    Ok(())

}
```

main函数使用了tokio异步处理.并在内部启动了两个独立的异步任务来处理两个阻塞性任务.p2p网络行为的loop和rocket监听.


在main中首先生成日志配置,监理日志记录文件,设置日志等级,初始化日志记录器

设定了两个日志记录器,终端和日志文件.

然后依据启动和初始化函数中获取的记录和环境变量初始化本地节点身份和监听端口.构建网络行为

然后启动了两个独立的异步任务来分别处理网络行为和rocket.

最后实现退出逻辑.

# 前端

## 前端重点

- 界面美观
- 远程访问方便
- 操作人性化
- 不同软件,平台和设备类型的兼容性优良
- 响应迅速
- 本地化

## 使用React构建的前端(部分未完成)

通过编写和组织不同模块来丰富功能,同时也保证了可读性,方便维护和扩展.
通过framer-motion, nextui, antd等库来优化使用体验

代码就省略了

构建后通过rocket嵌入部署,安全且部署便捷.