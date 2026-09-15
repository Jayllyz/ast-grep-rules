# ast-grep-rules

A collection of [ast-grep](https://ast-grep.github.io/) rules for enforcing JPA entity and Postgres naming conventions in Java code.

## Rules

- **entity-requires-table-name** — flags `@Entity` classes missing an explicit `@Table(name = "...")` annotation.
- **postgres-identifier-case** — flags mixed-case string literals in `@Table`, `@Column`, or `@JoinColumn` names, since Postgres folds unquoted identifiers to lowercase.
- **bigdecimal-requires-precision-scale** — flags `BigDecimal` fields missing `@Column(precision = ..., scale = ...)`, since Hibernate otherwise falls back to a default scale that can round differently than the Postgres `numeric(p,s)` column.
- **enumerated-requires-string** — flags `@Enumerated` without `EnumType.STRING`, since the default `EnumType.ORDINAL` persists the declaration order as an integer and silently corrupts data if enum constants are ever reordered, inserted, or removed.

## Using these rules in another repo

1. Install the ast-grep CLI:

   ```bash
   bun install -g @ast-grep/cli
   # or: npm install -g @ast-grep/cli / brew install ast-grep
   ```

2. Copy the `rules/` directory from this repo into your project (e.g. as a git submodule, or just copy the `.yml` files you need).

3. Point your project's `sgconfig.yml` at that directory:

   ```yaml
   ruleDirs:
     - path/to/rules
   ```

4. Run a scan:

   ```bash
   ast-grep scan --config sgconfig.yml
   ```

You can also run a single rule ad hoc without a config file:

```bash
ast-grep scan --rule path/to/rules/entity-requires-table-name.yml .
```

## Testing rules in this repo

Rule tests live in `rule-tests/` and are run with:

```bash
ast-grep test --config sgconfig.yml
```
