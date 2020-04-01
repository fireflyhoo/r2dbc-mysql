# Reactive Relational Database Connectivity MySQL Implementation

[![Maven Central](https://img.shields.io/maven-central/v/dev.miku/r2dbc-mysql?color=green&label=Maven%20Central)](https://search.maven.org/search?q=g:%22dev.miku%22%20AND%20a:%22r2dbc-mysql%22)
[![Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Unit tests status](https://github.com/mirromutth/r2dbc-mysql/workflows/Unit%20tests/badge.svg)](https://github.com/mirromutth/r2dbc-mysql/actions?query=workflow%3A%22Unit+tests%22)

This project contains the [MySQL][m] implementation of the [R2DBC SPI](https://github.com/r2dbc/r2dbc-spi).
This implementation is not intended to be used directly, but rather to be
used as the backing implementation for a humane client library to
delegate to. See [R2DBC Homepage](https://r2dbc.io).

This driver provides the following features:

- [x] Unix domain socket.
- [x] Execution of simple or batch statements without bindings.
- [x] Execution of prepared statements with bindings.
- [x] Reactive LOB types (e.g. BLOB, CLOB)
- [x] All charsets from MySQL, like `utf8mb4_0900_ai_ci`, `latin1_general_ci`, `utf32_unicode_520_ci`, etc.
- [x] All authentication types for MySQL, like `caching_sha2_password`, `mysql_native_password`, etc.
- [x] General exceptions of error code and standard SQL state mappings.
- [x] Secure connection with verification (SSL/TLS), auto-select TLS version for community and enterprise editions.
- [x] Transactions with savepoint.
- [x] Native ping command that can be verifying when argument is `ValidationDepth.REMOTE`
- [ ] Prepared statements cache.
- [ ] Statement parser cache.

## Version compatibility / Integration tests states

[![MySQL 5.5 status](https://github.com/mirromutth/r2dbc-mysql/workflows/MySQL%205.5/badge.svg)](https://github.com/mirromutth/r2dbc-mysql/actions?query=workflow%3A%22MySQL+5.5%22)
[![MySQL 5.6 status](https://github.com/mirromutth/r2dbc-mysql/workflows/MySQL%205.6/badge.svg)](https://github.com/mirromutth/r2dbc-mysql/actions?query=workflow%3A%22MySQL+5.6%22)
[![MySQL 5.7 status](https://github.com/mirromutth/r2dbc-mysql/workflows/MySQL%205.7/badge.svg)](https://github.com/mirromutth/r2dbc-mysql/actions?query=workflow%3A%22MySQL+5.7%22)
[![MySQL 8.0 status](https://github.com/mirromutth/r2dbc-mysql/workflows/MySQL%208.0/badge.svg)](https://github.com/mirromutth/r2dbc-mysql/actions?query=workflow%3A%22MySQL+8.0%22)

In fact, it supports lower versions, in the theory, such as 4.1, 4.0, etc.

But, Docker-certified images do not have these versions lower than 5.5.0, so tests are not integrated on these versions.

## Maven

```xml
<dependency>
    <groupId>dev.miku</groupId>
    <artifactId>r2dbc-mysql</artifactId>
    <version>0.8.1.RELEASE</version>
</dependency>
```

If you'd rather like the latest snapshots of the upcoming major version, use SonaType Maven snapshot repository and declare the appropriate dependency version.

```xml
<dependency>
    <groupId>dev.miku</groupId>
    <artifactId>r2dbc-mysql</artifactId>
    <version>${r2dbc-mysql.version}.BUILD-SNAPSHOT</version>
</dependency>

<repository>
    <id>sonatype-snapshots</id>
    <name>SonaType Snapshots</name>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    <snapshots>
        <enabled>true</enabled>
    </snapshots>
</repository>
```

## Gradle

### Groovy DSL

```groovy
dependencies {
    implementation 'dev.miku:r2dbc-mysql:0.8.1.RELEASE'
}
```

### Kotlin DSL

```kotlin
dependencies {
    // Maybe should to use `compile` instead of `implementation` on the lower version of Gradle.
    implementation("dev.miku:r2dbc-mysql:0.8.1.RELEASE")
}
```

## Getting Started

Here is a quick teaser of how to use R2DBC MySQL in Java:

### URL Connection Factory Discovery

```java
// Notice: the query string must be URL encoded
ConnectionFactory connectionFactory = ConnectionFactories.get(
    "r2dbcs:mysql://root:database-password-in-here@127.0.0.1:3306/r2dbc?" +
    "zeroDate=use_round&" +
    "sslMode=verify_identity&" +
    "useServerPrepareStatement=true&" +
    "tlsVersion=TLSv1.1%2CTLSv1.2%2CTLSv1.3&" +
    "sslCa=%2Fpath%2Fto%2Fmysql%2Fca.pem&" +
    "sslKey=%2Fpath%2Fto%2Fmysql%2Fclient-key.pem&" +
    "sslCert=%2Fpath%2Fto%2Fmysql%2Fclient-cert.pem&" +
    "sslKeyPassword=key-pem-password-in-here"
)

// Creating a Mono using Project Reactor
Mono<Connection> connectionMono = Mono.from(connectionFactory.create());
```

Or use unix domain socket like following:

```java
// Minimum configuration for unix domain socket
ConnectionFactory connectionFactory = ConnectionFactories.get("r2dbc:mysql://root@unix?unixSocket=%2Fpath%2Fto%2Fmysql.sock")

Mono<Connection> connectionMono = Mono.from(connectionFactory.create());
```

### Programmatic Connection Factory Discovery

```java
ConnectionFactoryOptions options = ConnectionFactoryOptions.builder()
    .option(DRIVER, "mysql")
    .option(HOST, "127.0.0.1")
    .option(USER, "root")
    .option(PORT, 3306)  // optional, default 3306
    .option(PASSWORD, "database-password-in-here") // optional, default null, null means has no password
    .option(DATABASE, "r2dbc") // optional, default null, null means not specifying the database
    .option(CONNECT_TIMEOUT, Duration.ofSeconds(3)) // optional, default null, null means no timeout
    .option(SSL, true) // optional, default is enabled, it will be ignore if "sslMode" is set
    .option(Option.valueOf("sslMode"), "verify_identity") // optional, default "preferred"
    .option(Option.valueOf("sslCa"), "/path/to/mysql/ca.pem") // required when sslMode is verify_ca or verify_identity, default null, null means has no server CA cert
    .option(Option.valueOf("sslKey"), "/path/to/mysql/client-key.pem") // optional, default null, null means has no client key
    .option(Option.valueOf("sslCert"), "/path/to/mysql/client-cert.pem") // optional, default null, null means has no client cert
    .option(Option.valueOf("sslKeyPassword"), "key-pem-password-in-here") // optional, default null, null means has no password for client key (i.e. "sslKey")
    .option(Option.valueOf("tlsVersion"), "TLSv1.1,TLSv1.2,TLSv1.3") // optional, default is auto-selected by the server
    .option(Option.valueOf("zeroDate"), "use_null") // optional, default "use_null"
    .option(Option.valueOf("useServerPrepareStatement"), true) // optional, default false
    .build();
ConnectionFactory connectionFactory = ConnectionFactories.get(options);

// Creating a Mono using Project Reactor
Mono<Connection> connectionMono = Mono.from(connectionFactory.create());
```

Or use unix domain socket like following:

```java
// Minimum configuration for unix domain socket
ConnectionFactoryOptions options = ConnectionFactoryOptions.builder()
    .option(DRIVER, "mysql")
    .option(Option.valueOf("unixSocket"), "/path/to/mysql.sock")
    .option(USER, "root")
    .build();
ConnectionFactory connectionFactory = ConnectionFactories.get(options);

Mono<Connection> connectionMono = Mono.from(connectionFactory.create());
```

### Programmatic Configuration

```java
MySqlConnectionConfiguration configuration = MySqlConnectionConfiguration.builder()
    .host("127.0.0.1")
    .username("root")
    .port(3306) // optional, default 3306
    .password("database-password-in-here") // optional, default null, null means has no password
    .database("r2dbc") // optional, default null, null means not specifying the database
    .connectTimeout(Duration.ofSeconds(3)) // optional, default null, null means no timeout
    .sslMode(SslMode.VERIFY_IDENTITY) // optional, default SslMode.PREFERRED
    .sslCa("/path/to/mysql/ca.pem") // required when sslMode is VERIFY_CA or VERIFY_IDENTITY, default null, null means has no server CA cert
    .sslKeyAndCert("/path/to/mysql/client-cert.pem", "/path/to/mysql/client-key.pem", "key-pem-password-in-here") // optional, default has no client key and cert
    .tlsVersion(TlsVersions.TLS1_1, TlsVersions.TLS1_2, TlsVersions.TLS1_3) // optional, default is auto-selected by the server
    .zeroDateOption(ZeroDateOption.USE_NULL) // optional, default ZeroDateOption.USE_NULL
    .useServerPrepareStatement() // Use server-preparing statements, default use client-preparing statements
    .build();
ConnectionFactory connectionFactory = MySqlConnectionFactory.from(configuration);

// Creating a Mono using Project Reactor
Mono<Connection> connectionMono = Mono.from(connectionFactory.create());
```

Or use unix domain socket like following:

```java
// Minimum configuration for unix domain socket
MySqlConnectionConfiguration configuration = MySqlConnectionConfiguration.builder()
    .unixSocket("/path/to/mysql.sock")
    .username("root")
    .build();
ConnectionFactory connectionFactory = MySqlConnectionFactory.from(configuration);

Mono<Connection> connectionMono = Mono.from(connectionFactory.create());
```

### Configuration items

| name | valid values | required | description |
|---|---|---|---|
| driver | A constant "mysql" | Required in R2DBC discovery | This driver needs to be discovered by name in R2DBC |
| host | A hostname or IP | Required when `unixSocket` does not exists | The host of MySQL database server |
| unixSocket | An absolute or relative path | Required when `host` does not exists | The `.sock` file of Unix Domain Socket |
| port | A positive integer less than 65536 | Optional, default 3306 | The port of MySQL database server |
| username | A valid MySQL username and not be empty | Required | Who wants to connect to the MySQL database |
| password | Any printable string | Optional, default no password | The password of the MySQL database user |
| database | A valid MySQL database name | Optional, default does not initialize database | Database used by the MySQL connection |
| connectTimeout | A `Duration` which must be positive duration | Optional, default has no timeout | TCP connect timeout |
| sslMode | Any value of `SslMode` | Optional, default `PREFERRED` when using hosting connection, `DISABLED` when using Unix Domain Socket | SSL mode, see following notice |
| sslCa | A path of local file which type is `PEM` | Required when `sslMode` is `VERIFY_CA` or `VERIFY_IDENTITY` | The CA cert of MySQL database server |
| sslCert | A path of local file which type is `PEM` | Required when `sslKey` exists | The SSL cert of client |
| sslKey | A path of local file which type is `PEM` | Required when `sslCert` exists | The SSL key of client |
| sslKeyPassword | Any valid password for `PEM` file | Optional, default `sslKey` has no password | The password for client SSL key (i.e. `sslKey`) |
| tlsVersion | Any value list of `TlsVersions` | Optional, default is auto-selected by the server | The TLS version for SSL, see following notice |
| zeroDateOption | Any value of `ZeroDateOption` | Optional, default `USE_NULL` | The option indicates "zero date" handling, see following notice |

- `SslMode`: Considers security level and verification for SSL, make sure the database server supports SSL before you want change SSL mode to `REQUIRED` or higher. **The Unix Domain Socket only offers "DISABLED" available**
  - `DISABLED`: I don't care about security and don't want to pay the overhead for encryption
  - `PREFERRED`: I don't care about encryption but will pay the overhead of encryption if the server supports it. **Unavailable on Unix Domain Socket**
  - `REQUIRED`: I want my data to be encrypted, and I accept the overhead. I trust that the network will make sure I always connect to the server I want. **Unavailable on Unix Domain Socket**
  - `VERIFY_CA`: I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server that I trust. **Unavailable on Unix Domain Socket**
  - `VERIFY_IDENTITY` (highest level, most like web browser): I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server I trust, and that it's the one I specify. **Unavailable on Unix Domain Socket**
- `TlsVersions`: Considers TLS version names for SSL, can be **multi-values** in configuration, make sure the database server supports selected TLS versions. **Unavailable on Unix Domain Socket**
  - `TLS1` (i.e. "TLSv1"): Under generic circumstances, MySQL database supports it if database supports SSL
  - `TLS1_1` (i.e. "TLSv1.1"): Under generic circumstances, MySQL database supports it if database supports SSL
  - `TLS1_2` (i.e. "TLSv1.2"): Supported only in Community Edition `8.0.4` or higher, otherwise in Enterprise Edition `5.6.0` or higher
  - `TLS1_3` (i.e. "TLSv1.3"): Supported only available as of MySQL `8.0.16` or higher, and requires compiling MySQL using OpenSSL `1.1.1` or higher
- `ZeroDateOption`: Considers special handling when MySQL database server returning "zero date" (i.e. `0000-00-00 00:00:00`)
  - `EXCEPTION`: Just throw a exception when MySQL database server return "zero date"
  - `USE_NULL`: Use `null` when MySQL database server return "zero date"
  - `USE_ROUND`: **NOT** RECOMMENDED, only for compatibility. Use "round" date (i.e. `0001-01-01 00:00:00`) when MySQL database server return "zero date"
- Prepare Statement: Considers based on server-preparing or client-preparing, some database server maybe not support server-preparing binary-query, such as Vitess
  - `useClientPrepareStatement()`: default preparing mode, use client-preparing text-query for parametrized statements
  - `useServerPrepareStatement()`: use server-preparing binary-query for parametrized statements
  - `useServerPrepareStatement(Predicate<String>)`: use server-preparing binary-query for parametrized statements, and enforce server-preparing usage for simple query (not parametrized statements). The usage is judged by `Predicate`, it's parameter is the simple SQL statement, enforce server-preparing if return `true`

Should use `enum` in [Programmatic](#programmatic-configuration) configuration that not like discovery configurations, except `TlsVersions` (All elements of `TlsVersions` will be always `String` which is case sensitive).

### Pooling

See [r2dbc-pool](https://github.com/r2dbc/r2dbc-pool).

### Simple statement

```java
connection.createStatement("INSERT INTO `person` (`first_name`, `last_name`) VALUES ('who', 'how')")
    .execute(); // return a Publisher include one Result
```

### Prepared statement

```java
connection.createStatement("INSERT INTO `person` (`birth`, `nickname`, `show_name`) VALUES (?, ?name, ?name)")
    .bind(0, LocalDateTime.of(2019, 6, 25, 12, 12, 12))
    .bind("name", "Some one") // Not one-to-one binding, call twice of native index-bindings, or call once of name-bindings.
    .add()
    .bind(0, LocalDateTime.of(2009, 6, 25, 12, 12, 12))
    .bind(1, "My Nickname")
    .bind(2, "Naming show")
    .returnGeneratedValues("generated_id")
    .execute(); // return a Publisher include two Results.
```

- All parameters must be bound before execute, even parameter is `null` (use `bindNull` to bind `null`).
- In one-to-one binding, because native MySQL prepared statements use index-based parameters, *index-bindings* will have **better** performance than *name-bindings*.

### Batch statement

```java
connection.createBatch()
    .add("INSERT INTO `person` (`first_name`, `last_name`) VALUES ('who', 'how')")
    .add("UPDATE `earth` SET `count` = `count` + 1 WHERE `id` = 'human'")
    .execute(); // return a Publisher include two Results.
```

> The last `;` will be removed if has only whitespace follow the last `;`.

### Transactions

```java
connection.beginTransaction()
    .then(Mono.from(connection.createStatement("INSERT INTO `person` (`first_name`, `last_name`) VALUES ('who', 'how')").execute()))
    .flatMap(Result::getRowsUpdated)
    .thenMany(connection.createStatement("INSERT INTO `person` (`birth`, `nickname`, `show_name`) VALUES (?, ?name, ?name)")
        .bind(0, LocalDateTime.of(2019, 6, 25, 12, 12, 12))
        .bind("name", "Some one")
        .add()
        .bind(0, LocalDateTime.of(2009, 6, 25, 12, 12, 12))
        .bind(1, "My Nickname")
        .bind(2, "Naming show")
        .returnGeneratedValues("generated_id")
        .execute())
    .flatMap(Result::getRowsUpdated)
    .then(connection.commitTransaction());
```

## Data Type Mapping

This reference table shows the type mapping between [MySQL][m] and Java data types:

| MySQL Type | Unsigned | Support Data Type |
|---|---|---|
| INT | UNSIGNED | [**`Long`**][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| INT | SIGNED | [**`Integer`**][java-Integer-ref], [`Long`][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| TINYINT | UNSIGNED | [**`Short`**][java-Short-ref], [`Integer`][java-Integer-ref], [`Long`][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| TINYINT | SIGNED | [**`Byte`**][java-Byte-ref], [`Short`][java-Short-ref], [`Integer`][java-Integer-ref], [`Long`][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| SMALLINT | UNSIGNED | [**`Integer`**][java-Integer-ref], [`Long`][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| SMALLINT | SIGNED | [**`Short`**][java-Short-ref], [`Integer`][java-Integer-ref], [`Long`][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| MEDIUMINT | SIGNED/UNSIGNED | [**`Integer`**][java-Integer-ref], [`Long`][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| BIGINT | UNSIGNED | [**`BigInteger`**][java-BigInteger-ref], [`Long`][java-Long-ref] (Not check overflow) |
| BIGINT | SIGNED | [**`Long`**][java-Long-ref], [`BigInteger`][java-BigInteger-ref] |
| FLOAT | SIGNED/UNSIGNED | [**`Float`**][java-Float-ref], [`BigDecimal`][java-BigDecimal-ref] |
| DOUBLE | SIGNED/UNSIGNED | [**`Double`**][java-Double-ref], [`BigDecimal`][java-BigDecimal-ref]  |
| DECIMAL | SIGNED/UNSIGNED | [**`BigDecimal`**][java-BigDecimal-ref], [`Float`][java-Float-ref] (Size less than 7), [`Double`][java-Double-ref] (Size less than 16) |
| BIT | - | [**`ByteBuffer`**][java-ByteBuffer-ref], [`BitSet`][java-BitSet-ref], [`Boolean`][java-Boolean-ref] (Size is 1), `byte[]` |
| DATETIME/TIMESTAMP | - | [**`LocalDateTime`**][java-LocalDateTime-ref] |
| DATE | - | [**`LocalDate`**][java-LocalDate-ref] |
| TIME | - | [**`LocalTime`**][java-LocalTime-ref], [`Duration`][java-Duration-ref] |
| YEAR | - | [**`Short`**][java-Short-ref], [`Integer`][java-Integer-ref], [`Long`][java-Long-ref], [`BigInteger`][java-BigInteger-ref], [`Year`][java-Year-ref] |
| VARCHAR/NVARCHAR | - | [**`String`**][java-String-ref] |
| VARBINARY | - | [**`ByteBuffer`**][java-ByteBuffer-ref], `Blob`, `byte[]` |
| CHAR/NCHAR | - | [**`String`**][java-String-ref] |
| ENUM | - | [**`String`**][java-String-ref], [`Enum<?>`][java-Enum-ref] |
| SET | - | **`String[]`**, [`String`][java-String-ref], [`Set<String>`][java-Set-ref] and [`Set<Enum<?>>`][java-Set-ref] ([`Set<T>`][java-Set-ref] need use [`ParameterizedType`][java-ParameterizedType-ref]) |
| BLOB (LONGBLOB, etc.) | - | [**`ByteBuffer`**][java-ByteBuffer-ref], `Blob`, `byte[]` |
| TEXT (LONGTEXT, etc.) | - | [**`String`**][java-String-ref], `Clob` |
| JSON | - | [**`String`**][java-String-ref], `Clob` |
| GEOMETRY | - | **`byte[]`**, `Blob` |

## Reporting Issues

The R2DBC MySQL Implementation uses GitHub as issue tracking system to record bugs and feature requests. 
If you want to raise an issue, please follow the recommendations below:

- Before you log a bug, please search the [issue tracker](https://github.com/mirromutth/r2dbc-mysql/issues) to see if someone has already reported the problem.
- If the issue doesn't already exist, [create a new issue](https://github.com/mirromutth/r2dbc-mysql/issues/new).
- Please provide as much information as possible with the issue report, we like to know the version of R2DBC MySQL that you are using and JVM version.
- If you need to paste code, or include a stack trace use Markdown **&#96;&#96;&#96;** escapes before and after your text.
- If possible try to create a test-case or project that replicates the issue. Attach a link to your code or a compressed file containing your code.

## Before use

- The MySQL data fields encoded by index-based natively, get fields by index will have **better** performance than get by column name.
- Each `Result` should be used (call `getRowsUpdated` or `map`, even table definition), can **NOT** just ignore any `Result`, otherwise inbound stream is unable to align. (like `ResultSet.close` in jdbc, `Result` auto-close after used by once)
- The MySQL returns microseconds &amp; milliseconds when only in server-preparing queries' result (and maybe contains not even in these results). Therefore this driver does not guarantee decoded `LocalDateTime`s contain more precision than seconds.
- The MySQL server does not **actively** return time zone when query `DATETIME` or `TIMESTAMP`, this driver does not attempt time zone conversion. That means should always use `LocalDateTime` for SQL type `DATETIME` or `TIMESTAMP`. Execute `SHOW VARIABLES LIKE '%time_zone%'` to get more information.
- Should not turn-on the `trace` log level unless debugging. Otherwise, the security information may be exposed through `ByteBuf` dump.
- If `Statement` bound `returnGeneratedValues`, the `Result` of the `Statement` can be called both: `getRowsUpdated` to get affected rows, and `map` to get last inserted ID.
- The MySQL may be not support search rows by binary field, like `BIT`, `BLOB` and `JSON`, because those data fields maybe just an address of reference in MySQL server, or maybe need strict bit-aligned. (but `VARBINARY` is OK)

## License

This project is released under version 2.0 of the [Apache License](https://www.apache.org/licenses/LICENSE-2.0).

[m]: https://www.mysql.com
[java-BigDecimal-ref]: https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html
[java-BigInteger-ref]: https://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html
[java-BitSet-ref]: https://docs.oracle.com/javase/8/docs/api/java/util/BitSet.html
[java-Boolean-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Boolean.html
[java-Byte-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Byte.html
[java-ByteBuffer-ref]: https://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html
[java-Double-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Double.html
[java-Float-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Float.html
[java-Integer-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html
[java-Long-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Long.html
[java-LocalDateTime-ref]: https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html
[java-LocalDate-ref]: https://docs.oracle.com/javase/8/docs/api/java/time/LocalDate.html
[java-Duration-ref]: https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html
[java-LocalTime-ref]: https://docs.oracle.com/javase/8/docs/api/java/time/LocalTime.html
[java-Year-ref]: https://docs.oracle.com/javase/8/docs/api/java/time/Year.html
[java-Short-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Short.html
[java-Enum-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html
[java-String-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/String.html
[java-Set-ref]: https://docs.oracle.com/javase/8/docs/api/java/util/Set.html
[java-ParameterizedType-ref]: https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/ParameterizedType.html
