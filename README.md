# ast-grep-rules

A collection of [ast-grep](https://ast-grep.github.io/) rules for Java: JPA/Postgres entity conventions plus general Java hygiene.

## Rules

See [`rules/`](rules) for the full rule definitions.

- [`entity-requires-table-name`](rules/entity-requires-table-name.yml): `@Entity` needs an explicit `@Table(name = "...")`.
- [`postgres-identifier-case`](rules/postgres-identifier-case.yml): no mixed-case table/column/join-column names.
- [`bigdecimal-requires-precision-scale`](rules/bigdecimal-requires-precision-scale.yml): `BigDecimal` fields need `@Column(precision, scale)`.
- [`enumerated-requires-string`](rules/enumerated-requires-string.yml): `@Enumerated` needs `EnumType.STRING`.
- [`to-many-requires-lazy-fetch`](rules/to-many-requires-lazy-fetch.yml): `@OneToMany`/`@ManyToMany` need explicit `fetch = FetchType.LAZY`.
- [`require-instant-or-offsetdatetime`](rules/require-instant-or-offsetdatetime.yml): no `java.util.Date`/`LocalDateTime`, use `Instant`/`OffsetDateTime`.
- [`find-by-id-requires-optional`](rules/find-by-id-requires-optional.yml): `findById`/`getById`-style repository methods must return `Optional<T>`.
- [`no-empty-catch-block`](rules/no-empty-catch-block.yml): no silently swallowed exceptions.
- [`collectors-tolist-to-stream-tolist`](rules/collectors-tolist-to-stream-tolist.yml): use `Stream.toList()` instead of `.collect(Collectors.toList())`.
- [`no-optional-field-or-parameter`](rules/no-optional-field-or-parameter.yml): `Optional` should only be used as a return type.
- [`no-boxed-type-constructor`](rules/no-boxed-type-constructor.yml): no `new Integer(...)`/`new Boolean(...)`/etc., use `valueOf()` or autoboxing.
- [`no-debug-print`](rules/no-debug-print.yml): no `System.out/err.println`/`printStackTrace()` outside test code, route through a logger instead.
- [`no-wildcard-import`](rules/no-wildcard-import.yml): no `import java.util.*;`, import the specific types used.
- [`no-public-mutable-field`](rules/no-public-mutable-field.yml): no public non-final fields on classes, encapsulate with an accessor.
- [`no-field-injection`](rules/no-field-injection.yml): no `@Autowired`/`@Inject` directly on a field, use constructor injection.
- [`sealed-switch-default-throw`](rules/sealed-switch-default-throw.yml): a pattern-matching `switch` with a throwing `default` usually means the type should be sealed.
- [`instanceof-pattern-variable`](rules/instanceof-pattern-variable.yml): use `if (x instanceof Foo f)` instead of a separate `instanceof` check plus cast.

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
