# RustDesk Server 编译警告记录

本文档记录了RustDesk Server项目编译过程中出现的警告信息，以便后续修复。

## 1. libs/hbb_common/src/proxy.rs

**警告类型**: 使用已弃用的函数

**行号**: 458

**警告内容**:
```
warning: use of deprecated function `rustls_platform_verifier::tls_config`: use the `ConfigVerifierExt` instead
   --> libs/hbb_common/src/proxy.rs:458:50
    |
458 |         let verifier = rustls_platform_verifier::tls_config();
    |                                                  ^^^^^^^^^^
```

**修复建议**: 使用`ConfigVerifierExt`替代`tls_config()`函数。

## 2. libs/hbb_common/src/websocket.rs

**警告类型**: 未使用的变量

**行号**: 33-34

**警告内容**:
```
warning: unused variable: `local_addr`
  --> libs/hbb_common/src/websocket.rs:33:9
   |
33 |         local_addr: Option<SocketAddr>
   |         ^^^^^^^^^^ help: if this is intentional prefix it with an underscore: `_local_addr`

warning: unused variable: `proxy_conf`
  --> libs/hbb_common/src/websocket.rs:34:9
   |
34 |         proxy_conf: Option<&Socks5Server>
   |         ^^^^^^^^^^ help: if this is intentional prefix it with an underscore: `_proxy_conf`
```

**修复建议**: 如果这些变量是有意不使用的，可以在变量名前加下划线前缀，如`_local_addr`和`_proxy_conf`。

## 3. libs/hbb_common/src/platform/mod.rs

**警告类型**: 创建对可变静态变量的共享引用

**行号**: 65

**警告内容**:
```
warning: creating a shared reference to mutable static is discouraged
  --> libs/hbb_common/src/platform/mod.rs:65:33
   |
65 |         if let Some(callback) = &GLOBAL_CALLBACK {
   |                                 ^^^^^^^^^^^^^^^^ shared reference to mutable static
   |
   = note: for more information see <https://doc.rust-lang.org/nightly/edition-guide/rust-2024/static-mut-references.html>
   = note: shared references to mutable statics are dangerous; it's undefined behavior if the static is mutated or if a mutable reference is created for it while the shared reference lives
```

**修复建议**: 使用`&raw const`代替创建原始指针，如下所示：
```rust
if let Some(callback) = &raw const GLOBAL_CALLBACK {
```

## 4. src/database.rs

**警告类型**: 未使用的字段

**行号**: 41, 44, 46

**警告内容**:
```
warning: fields `id` `user` and `status` are never read
  --> src/database.rs:41:9
   |
39 | pub struct Peer {
   |            ---- fields in this struct
40 |     pub guid: Vec<u8>
41 |     pub id: String
   |         ^^
...
44 |     pub user: Option<Vec<u8>>
   |         ^^^^
45 |     pub info: String
46 |     pub status: Option<i64>
   |         ^^^^^^
```

**修复建议**: 如果这些字段确实不需要，可以考虑移除它们；如果它们在将来可能会使用，可以添加`#[allow(dead_code)]`属性到结构体上。

## 5. 未来不兼容性警告

**警告类型**: 未来版本的Rust将拒绝的代码

**警告内容**:
```
warning: the following packages contain code that will be rejected by a future version of Rust: nom v5.1.2
note: to see what the problems were use the option `--future-incompat-report` or run `cargo report future-incompatibilities --id 1`
```

**修复建议**: 运行`cargo report future-incompatibilities --id 1`查看详细问题，并考虑更新`nom`包到更新的版本。

## 总结

项目编译过程中共出现了5类警告，涉及4个文件。这些警告虽然不影响当前的编译和运行，但可能会在未来的Rust版本中导致问题，建议及时修复。
