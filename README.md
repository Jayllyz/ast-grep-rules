# ast-grep-rules

A collection of [ast-grep](https://ast-grep.github.io/) rules for Java.

## Rules

See [`rules/`](rules) for the full rule definitions, grouped into `jpa/`, `performance/`, and `java/` subdirectories. Every rule also carries a `metadata.category` matching its directory.

### JPA / persistence

- [`entity-requires-table-name`](rules/jpa/entity-requires-table-name.yml): `@Entity` needs an explicit `@Table(name = "...")`.
- [`postgres-identifier-case`](rules/jpa/postgres-identifier-case.yml): no mixed-case table/column/join-column names.
- [`bigdecimal-requires-precision-scale`](rules/jpa/bigdecimal-requires-precision-scale.yml): `BigDecimal` fields need `@Column(precision, scale)`.
- [`enumerated-requires-string`](rules/jpa/enumerated-requires-string.yml): `@Enumerated` needs `EnumType.STRING`.
- [`to-many-requires-lazy-fetch`](rules/jpa/to-many-requires-lazy-fetch.yml): `@OneToMany`/`@ManyToMany` need explicit `fetch = FetchType.LAZY`.
- [`to-one-requires-lazy-fetch`](rules/jpa/to-one-requires-lazy-fetch.yml): `@ManyToOne`/`@OneToOne` need explicit `fetch = FetchType.LAZY` (they default to `EAGER`).
- [`require-instant-or-offsetdatetime`](rules/jpa/require-instant-or-offsetdatetime.yml): no `java.util.Date`/`LocalDateTime`, use `Instant`/`OffsetDateTime`.
- [`no-primitive-jpa-id`](rules/jpa/no-primitive-jpa-id.yml): `@Id` fields must use a boxed type, not `long`/`int`.
- [`generated-value-requires-strategy`](rules/jpa/generated-value-requires-strategy.yml): `@GeneratedValue` needs an explicit, non-`AUTO` strategy.
- [`join-column-requires-name`](rules/jpa/join-column-requires-name.yml): `@JoinColumn` needs an explicit `name = "..."`.
- [`find-by-id-requires-optional`](rules/jpa/find-by-id-requires-optional.yml): `findById`/`getById`-style repository methods must return `Optional<T>`.
- [`no-lombok-data-on-entity`](rules/jpa/no-lombok-data-on-entity.yml): no Lombok `@Data`/bare `@EqualsAndHashCode` on a JPA `@Entity`.
- [`no-query-string-concatenation`](rules/jpa/no-query-string-concatenation.yml): no `@Query("..." + x)`, use a constant query with parameters.
- [`no-nplus1-call-in-loop`](rules/jpa/no-nplus1-call-in-loop.yml): no repository query inside a loop (the N+1 problem), fetch the batch up front.
- [`no-save-in-loop`](rules/jpa/no-save-in-loop.yml): no `save`/`saveAndFlush`/`persist`/`merge` per loop iteration, batch with `saveAll`.
- [`prefer-exists-over-count`](rules/jpa/prefer-exists-over-count.yml): no `count(...) > 0` existence checks, use `exists(...)`/`existsBy...`.
- [`no-findall-for-count-or-filter`](rules/jpa/no-findall-for-count-or-filter.yml): no `findAll().size()`/`isEmpty()`/`stream().filter(...)`, use `count()`/`existsBy...`/a derived query.
- [`many-to-many-requires-set`](rules/jpa/many-to-many-requires-set.yml): `@ManyToMany` needs a `Set`, a `List` makes Hibernate rewrite the whole join table.
- [`transactional-requires-proxyable-method`](rules/jpa/transactional-requires-proxyable-method.yml): `@Transactional` on a `private`/`static`/`final` method is silently ignored.
- [`no-repository-or-else-null`](rules/jpa/no-repository-or-else-null.yml): no `repo.findX(...).orElse(null)`, use `orElseThrow(...)` or handle the `Optional`.
- [`sort-by-requires-typed-property`](rules/jpa/sort-by-requires-typed-property.yml): no `Sort.by("field")` string properties, use `Sort.sort(Entity.class).by(Entity::getField)`.
- [`criteria-requires-metamodel`](rules/jpa/criteria-requires-metamodel.yml): no string attributes in Criteria API `root.get("x")`/`join("x")`, use the static metamodel (`Order_.x`).
- [`one-to-many-requires-mappedby`](rules/jpa/one-to-many-requires-mappedby.yml): `@OneToMany` needs `mappedBy`, otherwise JPA adds a join table.
- [`read-method-transactional-readonly`](rules/jpa/read-method-transactional-readonly.yml): `@Transactional` read methods should set `readOnly = true`.

### Performance

- [`collectors-tolist-to-stream-tolist`](rules/performance/collectors-tolist-to-stream-tolist.yml): use `Stream.toList()` instead of `.collect(Collectors.toList())`.
- [`prefer-entryset-over-keyset`](rules/performance/prefer-entryset-over-keyset.yml): no `keySet()` + `get(key)`, iterate `entrySet()` instead.
- [`no-string-matches`](rules/performance/no-string-matches.yml): no `String.matches`/`replaceAll`/`replaceFirst`/`split`, reuse a precompiled `Pattern`.
- [`no-regex-compile-in-loop`](rules/performance/no-regex-compile-in-loop.yml): don't `Pattern.compile(...)` inside a loop, hoist it to a `static final` field.
- [`no-simpledateformat-in-loop`](rules/performance/no-simpledateformat-in-loop.yml): don't build a `SimpleDateFormat` inside a loop, hoist it out.
- [`no-stringbuffer`](rules/performance/no-stringbuffer.yml): no `new StringBuffer(...)`, use `StringBuilder` to avoid per-operation synchronization.
- [`parameterized-logging`](rules/performance/parameterized-logging.yml): use `log.debug("x={}", x)` instead of concatenating arguments into a log message.
- [`or-else-eager-default`](rules/performance/or-else-eager-default.yml): no `orElse(expensive())`, use `orElseGet(...)` so the default is lazy.
- [`no-static-simpledateformat`](rules/performance/no-static-simpledateformat.yml): no static `SimpleDateFormat`, it is not thread-safe.

### Java hygiene & correctness

- [`obvious-comment`](rules/java/obvious-comment.yml): no comments that restate what the code already says (`// increment the counter`).
- [`controller-endpoint-requires-operation`](rules/java/controller-endpoint-requires-operation.yml): every `@*Mapping` handler needs `@Operation`/`@ApiOperation` OpenAPI docs.
- [`no-empty-catch-block`](rules/java/no-empty-catch-block.yml): no silently swallowed exceptions.
- [`no-catch-throwable`](rules/java/no-catch-throwable.yml): don't catch `Throwable`/`Error`.
- [`no-return-in-finally`](rules/java/no-return-in-finally.yml): no `return`/`throw` inside a `finally` block.
- [`no-finalize-method`](rules/java/no-finalize-method.yml): no `Object.finalize()` overrides.
- [`no-optional-get`](rules/java/no-optional-get.yml): no bare `Optional.get()`, prefer `orElseThrow()`/`orElse()`.
- [`no-optional-field-or-parameter`](rules/java/no-optional-field-or-parameter.yml): `Optional` should only be used as a return type.
- [`no-field-injection`](rules/java/no-field-injection.yml): no `@Autowired`/`@Inject` directly on a field, use constructor injection.
- [`no-public-mutable-field`](rules/java/no-public-mutable-field.yml): no public non-final fields on classes, encapsulate with an accessor.
- [`no-legacy-synchronized-collection`](rules/java/no-legacy-synchronized-collection.yml): no `new Vector()`/`Stack()`/`Hashtable()`.
- [`no-raw-type-instantiation`](rules/java/no-raw-type-instantiation.yml): no raw `new ArrayList()`/`HashMap()`/etc.
- [`no-boxed-type-constructor`](rules/java/no-boxed-type-constructor.yml): no `new Integer(...)`/`new Boolean(...)`/etc., use `valueOf()` or autoboxing.
- [`no-new-string`](rules/java/no-new-string.yml): no `new String(...)`, use the literal/argument directly.
- [`no-bigdecimal-double-constructor`](rules/java/no-bigdecimal-double-constructor.yml): no `new BigDecimal(double)`, use `valueOf` or the `String` ctor.
- [`prefer-isempty`](rules/java/prefer-isempty.yml): use `isEmpty()` instead of `size()/length() == 0` or `equals("")`.
- [`instanceof-pattern-variable`](rules/java/instanceof-pattern-variable.yml): use `if (x instanceof Foo f)` instead of a separate `instanceof` check plus cast.
- [`sealed-switch-default-throw`](rules/java/sealed-switch-default-throw.yml): a pattern-matching `switch` with a throwing `default` usually means the type should be sealed.
- [`no-wildcard-import`](rules/java/no-wildcard-import.yml): no `import java.util.*;`, import the specific types used.
- [`no-debug-print`](rules/java/no-debug-print.yml): no `System.out/err.println`/`printStackTrace()` outside test code, route through a logger instead.
- [`no-system-gc`](rules/java/no-system-gc.yml): no `System.gc()`/`Runtime.getRuntime().gc()`.
- [`no-runtime-exec`](rules/java/no-runtime-exec.yml): no `Runtime.getRuntime().exec(...)`, use `ProcessBuilder`.
- [`no-java-deserialization`](rules/java/no-java-deserialization.yml): no `ObjectInputStream.readObject()`/`readUnshared()` on untrusted bytes, a remote-code-execution vector.
- [`equals-without-hashcode`](rules/java/equals-without-hashcode.yml): overriding `equals` requires a matching `hashCode`.
- [`no-catch-nullpointer`](rules/java/no-catch-nullpointer.yml): don't catch `NullPointerException`, fix the null source.
- [`no-thread-stop`](rules/java/no-thread-stop.yml): no `Thread.stop()`/`suspend()`/`resume()`, interrupt instead.
- [`no-locale-sensitive-case`](rules/java/no-locale-sensitive-case.yml): `toLowerCase()`/`toUpperCase()` need an explicit `Locale`.

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
ast-grep scan --rule rules/jpa/entity-requires-table-name.yml .
```

## Running with an agent

This repo ships an agent skill at [`.agents/skills/run-ast-grep-rules`](.agents/skills/run-ast-grep-rules/SKILL.md), using the cross-tool `.agents/skills/` convention. When the repo is present (or its rules are copied into a project alongside `.agents/`), an agent can run the rules over a codebase: it asks which categories to run (all by default) and returns a minimal report.

## Running only on staged files

To scan just the Java files staged for commit (e.g. from a pre-commit hook), pass their paths straight from `git diff`:

```bash
git diff --staged --name-only --diff-filter=ACM -- '*.java' | xargs -r ast-grep scan --config sgconfig.yml
```

`--diff-filter=ACM` skips deleted/renamed files so `ast-grep` isn't asked to scan a path that no longer exists, and `xargs -r` avoids running `ast-grep` at all when nothing staged matches.

## Pre-commit hook with Lefthook

[Lefthook](https://github.com/evilmartians/lefthook) can run the rules on staged files automatically before each commit.

1. Install Lefthook (see the [Lefthook docs](https://lefthook.dev/installation.html) for other install methods):

   ```bash
   npm install -D lefthook
   # or: brew install lefthook / go install github.com/evilmartians/lefthook@latest
   ```

2. Add a `lefthook.yml` at your project root:

   ```yaml
   pre-commit:
     commands:
       ast-grep:
         glob: "*.java"
         run: ast-grep scan --config sgconfig.yml {staged_files}
   ```

3. Install the git hooks:

   ```bash
   lefthook install
   ```

Now `git commit` runs `ast-grep` against only the staged `.java` files, and the commit is blocked if any `error`-severity rule fires.

## Testing rules in this repo

Rule tests live in `rule-tests/` and are run with:

```bash
ast-grep test --config sgconfig.yml
```
