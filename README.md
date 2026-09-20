# Termini fork of crates.io `rig-agent` 0.42.0

只记跟官方 **0.42.0** 源码的差异。基底是 crates.io `.crate`，不是 git 快照。

Termini 挂载：

```toml
[patch.crates-io]
rig-core = { path = "../../archive/rig" }
rig-agent = { path = "../../archive/rig-agent" }
```

`rig` 门面只做 re-export，不 patch。第 2、3、4 处在 `archive/rig`（`rig-core`）。

---

## 1. 诊断 tool call 空参数 `Null` → `{}`

文件：`src/agent/run/streamed.rs`  
函数：`name_delta_diagnostic_tool_call`

模型先吐工具名、后吐参数时，rig 会先造一条诊断调用。官方把空参数或解析失败记成 `Null`。Skip / Retry 回放进历史后，下一跳由 rig 自己发出，OpenAI 兼容路径把 `Null` 序列化成 `"arguments":"null"`，百炼 400。

```rust
let diagnostic_args = if buffered_args.trim().is_empty() {
    serde_json::Value::Object(serde_json::Map::new())
} else {
    serde_json::from_str(buffered_args)
        .unwrap_or_else(|_| serde_json::Value::Object(serde_json::Map::new()))
};
```

这是官方源码里唯一产出 `Null` 参数的位置。

测例：`streamed_invalid_name_delta_without_args_records_object_arguments`、`streamed_invalid_name_delta_unparseable_args_normalize_to_object`。

---

## 不在本仓库

`stringified_json::serialize` 的 `Null` → `"{}"`、流式 `delta.images`、`rustls-no-provider` 在 `archive/rig`。

`.cargo_vcs_info.json`、`Cargo.toml.orig` 是 crates.io 包装文件，不是补丁。`Cargo.lock` 不是补丁。
