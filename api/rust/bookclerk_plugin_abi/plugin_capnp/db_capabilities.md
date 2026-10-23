# bookclerk_plugin_abi::plugin_capnp::db_capabilities

| Item | Kind | Summary |
| --- | --- | --- |
| [`Owned`](#owned) | struct |  |
| [`Reader`](#reader) | struct |  |
| [`Builder`](#builder) | struct |  |
| [`Pipeline`](#pipeline) | struct |  |

## Structs

### Owned

```rust
pub struct Owned(/* private */);
```

Implements: `Clone`, `Copy`, `Introspect`, `Owned`, `OwnedStruct`, `Pipelined`, `SetterInput<Owned>`

### Reader

```rust
pub struct Reader<'a> {
    /* private fields */
}
```

#### Reader::reborrow

```rust
pub fn reborrow(&self) -> Reader<'_>
```

#### Reader::total_size

```rust
pub fn total_size(&self) -> Result<MessageSize>
```

#### Reader::get_sql_contract_version

```rust
pub fn get_sql_contract_version(self) -> u32
```

#### Reader::get_atomic_batch

```rust
pub fn get_atomic_batch(self) -> bool
```

#### Reader::get_returning

```rust
pub fn get_returning(self) -> bool
```

#### Reader::get_affected_rows

```rust
pub fn get_affected_rows(self) -> bool
```

#### Reader::get_schema_migrations

```rust
pub fn get_schema_migrations(self) -> bool
```

#### Reader::get_cancellation

```rust
pub fn get_cancellation(self) -> bool
```

#### Reader::get_timing

```rust
pub fn get_timing(self) -> bool
```

#### Reader::get_max_binds

```rust
pub fn get_max_binds(self) -> u32
```

#### Reader::get_max_statements

```rust
pub fn get_max_statements(self) -> u32
```

#### Reader::get_max_result_rows

```rust
pub fn get_max_result_rows(self) -> u32
```

#### Reader::get_max_payload_bytes

```rust
pub fn get_max_payload_bytes(self) -> u32
```

#### Reader::get_max_result_bytes

```rust
pub fn get_max_result_bytes(self) -> u32
```

#### Reader::get_max_cell_bytes

```rust
pub fn get_max_cell_bytes(self) -> u32
```

#### Reader::get_max_request_bytes

```rust
pub fn get_max_request_bytes(self) -> u32
```

#### Reader::get_max_atomic_result_bytes

```rust
pub fn get_max_atomic_result_bytes(self) -> u32
```

#### Reader::get_plugin_databases

```rust
pub fn get_plugin_databases(self) -> bool
```

#### Reader::get_max_function_args

```rust
pub fn get_max_function_args(self) -> u32
```

#### Reader::get_max_schema_columns

```rust
pub fn get_max_schema_columns(self) -> u32
```

#### Reader::get_max_pattern_bytes

```rust
pub fn get_max_pattern_bytes(self) -> u32
```

#### Reader::get_max_lowered_statement_bytes

```rust
pub fn get_max_lowered_statement_bytes(self) -> u32
```

#### Reader::get_consistent_backup_read

```rust
pub fn get_consistent_backup_read(self) -> bool
```

#### Reader::get_atomic_unit_restore

```rust
pub fn get_atomic_unit_restore(self) -> bool
```

Implements: `Clone`, `Copy`, `Debug`, `From<Reader<'a>>`, `From<StructReader<'a>>`, `FromPointerReader<'a>`, `HasTypeId`, `Imbue<'a>`, `IntoInternalStructReader<'a>`, `SetterInput<Owned>`

### Builder

```rust
pub struct Builder<'a> {
    /* private fields */
}
```

#### Builder::into_reader

```rust
pub fn into_reader(self) -> Reader<'a>
```

#### Builder::reborrow

```rust
pub fn reborrow(&mut self) -> Builder<'_>
```

#### Builder::reborrow_as_reader

```rust
pub fn reborrow_as_reader(&self) -> Reader<'_>
```

#### Builder::total_size

```rust
pub fn total_size(&self) -> Result<MessageSize>
```

#### Builder::get_sql_contract_version

```rust
pub fn get_sql_contract_version(self) -> u32
```

#### Builder::set_sql_contract_version

```rust
pub fn set_sql_contract_version(&mut self, value: u32)
```

#### Builder::get_atomic_batch

```rust
pub fn get_atomic_batch(self) -> bool
```

#### Builder::set_atomic_batch

```rust
pub fn set_atomic_batch(&mut self, value: bool)
```

#### Builder::get_returning

```rust
pub fn get_returning(self) -> bool
```

#### Builder::set_returning

```rust
pub fn set_returning(&mut self, value: bool)
```

#### Builder::get_affected_rows

```rust
pub fn get_affected_rows(self) -> bool
```

#### Builder::set_affected_rows

```rust
pub fn set_affected_rows(&mut self, value: bool)
```

#### Builder::get_schema_migrations

```rust
pub fn get_schema_migrations(self) -> bool
```

#### Builder::set_schema_migrations

```rust
pub fn set_schema_migrations(&mut self, value: bool)
```

#### Builder::get_cancellation

```rust
pub fn get_cancellation(self) -> bool
```

#### Builder::set_cancellation

```rust
pub fn set_cancellation(&mut self, value: bool)
```

#### Builder::get_timing

```rust
pub fn get_timing(self) -> bool
```

#### Builder::set_timing

```rust
pub fn set_timing(&mut self, value: bool)
```

#### Builder::get_max_binds

```rust
pub fn get_max_binds(self) -> u32
```

#### Builder::set_max_binds

```rust
pub fn set_max_binds(&mut self, value: u32)
```

#### Builder::get_max_statements

```rust
pub fn get_max_statements(self) -> u32
```

#### Builder::set_max_statements

```rust
pub fn set_max_statements(&mut self, value: u32)
```

#### Builder::get_max_result_rows

```rust
pub fn get_max_result_rows(self) -> u32
```

#### Builder::set_max_result_rows

```rust
pub fn set_max_result_rows(&mut self, value: u32)
```

#### Builder::get_max_payload_bytes

```rust
pub fn get_max_payload_bytes(self) -> u32
```

#### Builder::set_max_payload_bytes

```rust
pub fn set_max_payload_bytes(&mut self, value: u32)
```

#### Builder::get_max_result_bytes

```rust
pub fn get_max_result_bytes(self) -> u32
```

#### Builder::set_max_result_bytes

```rust
pub fn set_max_result_bytes(&mut self, value: u32)
```

#### Builder::get_max_cell_bytes

```rust
pub fn get_max_cell_bytes(self) -> u32
```

#### Builder::set_max_cell_bytes

```rust
pub fn set_max_cell_bytes(&mut self, value: u32)
```

#### Builder::get_max_request_bytes

```rust
pub fn get_max_request_bytes(self) -> u32
```

#### Builder::set_max_request_bytes

```rust
pub fn set_max_request_bytes(&mut self, value: u32)
```

#### Builder::get_max_atomic_result_bytes

```rust
pub fn get_max_atomic_result_bytes(self) -> u32
```

#### Builder::set_max_atomic_result_bytes

```rust
pub fn set_max_atomic_result_bytes(&mut self, value: u32)
```

#### Builder::get_plugin_databases

```rust
pub fn get_plugin_databases(self) -> bool
```

#### Builder::set_plugin_databases

```rust
pub fn set_plugin_databases(&mut self, value: bool)
```

#### Builder::get_max_function_args

```rust
pub fn get_max_function_args(self) -> u32
```

#### Builder::set_max_function_args

```rust
pub fn set_max_function_args(&mut self, value: u32)
```

#### Builder::get_max_schema_columns

```rust
pub fn get_max_schema_columns(self) -> u32
```

#### Builder::set_max_schema_columns

```rust
pub fn set_max_schema_columns(&mut self, value: u32)
```

#### Builder::get_max_pattern_bytes

```rust
pub fn get_max_pattern_bytes(self) -> u32
```

#### Builder::set_max_pattern_bytes

```rust
pub fn set_max_pattern_bytes(&mut self, value: u32)
```

#### Builder::get_max_lowered_statement_bytes

```rust
pub fn get_max_lowered_statement_bytes(self) -> u32
```

#### Builder::set_max_lowered_statement_bytes

```rust
pub fn set_max_lowered_statement_bytes(&mut self, value: u32)
```

#### Builder::get_consistent_backup_read

```rust
pub fn get_consistent_backup_read(self) -> bool
```

#### Builder::set_consistent_backup_read

```rust
pub fn set_consistent_backup_read(&mut self, value: bool)
```

#### Builder::get_atomic_unit_restore

```rust
pub fn get_atomic_unit_restore(self) -> bool
```

#### Builder::set_atomic_unit_restore

```rust
pub fn set_atomic_unit_restore(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
