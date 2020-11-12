---
tags:
- Java
---

# Google Guava をもっと活用する

## Google Guava とは

[Guava](https://github.com/google/guava) とは Google が開発している Java の Common ライブラリ。

先日、[最新バージョン v19.0](https://github.com/google/guava/wiki/Release19)もリリースされました。

非常に便利なライブラリなので、使っている人たちも多いはず。面倒なロジックを組まないといけない場合は、とりあえず使える機能がないか調べるようにしています。

Java8 が出てきて、[FluentIterable](https://github.com/google/guava/wiki/CollectionUtilitiesExplained#fluentiterable) など競合する機能や出番の少なくなった機能も多いですが、使える機能はたくさんありますので、紹介してみようと思います。使うときに不便だった点も付け加えておきます。

## [Table](https://github.com/google/guava/wiki/NewCollectionTypesExplained#table)

* キーを2種類持てる Map のようなもの。
* 複数のキーで(といっても2種類)オブジェクト集合を検索するための一時的な索引として使うと便利。
* 2種類があれば3種類も、と思うが、3種類以上のキーを持てるものは、残念ながら無い。

```java
List<Employee> employees = ...;
... employees 作成 ...
Table<Integer, String, Employee> table = HashBasedTable.create();
employees.stream().forEach(employee -> {
	table.put(employee.getNo(), employee.getName(), employee);
});
---
private static class Employee {
	private int no;
	private String name;
	:
	public int getNo() {
		return no;
	}
	public String getName() {
		return name;
	}
	:
}
```

## [Multimap](https://github.com/google/guava/wiki/NewCollectionTypesExplained#multimap)

* 同じキーに対して複数の値を持つことができる。
* データの持ち方は `a -> 1, a -> 2, b -> 3, c -> 2`。リストを値に持つ `Map` ではない。
* なので、キーに対して値は一つという仕様の `Map` を継承することはリスコフの置換原則に違反するので、なんで `Map` を継承していないのだという文句は言わない。[Map との実装差分](https://github.com/google/guava/wiki/NewCollectionTypesExplained#multimap-is-not-a-map)もある。
* というところで、実はあまり使ったことがない。

```java
Multimap<String, String> m = HashMultimap.create();
m.put("a", "1");
m.put("a", "2");
m.put("b", "3");
m.put("c", "4");

m.get("a"); // [1, 2]
m.get("d"); // [] (キーがない場合は空のコレクション)

Map<String, Collection<String>> map = m.asMap();	// Mapへ変換
```

## [Range](https://github.com/google/guava/wiki/RangesExplained)

* 値の範囲を表現するクラス。
* 範囲をつなげたり共通部分を見つけたりするメソッドもある。
* `Comparable` な値であればこのクラスで表現可能なので、`java.time` との組み合わせることで期間を表現できたり、使い方次第でいろいろ使える。
* また、[RangeSet](https://github.com/google/guava/wiki/NewCollectionTypesExplained#rangeset), [RangeMap](https://github.com/google/guava/wiki/NewCollectionTypesExplained#rangemap) というコレクションも用意されているので、そちらと組み合わせるも便利。
* `DiscreteDomain` と `ContiguousSet` との組み合わせで、連続する範囲 (`[1,2,3]` とか `[5,6,...,Integer.MAX_VALUE]` とか) を生成することができるらしいが、Java8 を使っていれば `IntStream` や `LongStream` を使うと作れるので、多分使わない。

```java
Range<MonthDay> summerVacation = Range.closed(MonthDay.of(7, 21), MonthDay.of(8, 31));
if (!summerVacation.contains(MonthDay.of(9, 1))) {
    System.out.println("夏休みは終わり");
}

RangeMap<LocalDate, String> primeMinisters = new ImmutableRangeMap.Builder<LocalDate, String>()
	.put(Range.closedOpen(LocalDate.of(2010, 6, 8), LocalDate.of(2011, 9, 2)), "管さん")
	.put(Range.closedOpen(LocalDate.of(2011, 9, 2), LocalDate.of(2012, 12, 26)), "野田さん")
	.put(Range.atLeast(LocalDate.of(2012, 12, 26)), "安倍さん")
	.build();
primeMinisters.get(LocalDate.now());	// 安倍さん
```

```java
for (int i : ContiguousSet.create(Range.open(0, 4), DiscreteDomain.integers())) {
    System.out.println(i);
}
// これと同じ
IntStream.range(0, 4).forEach(System.out::println);
// 1
// 2
// 3
```

## [Hash](https://github.com/google/guava/wiki/HashingExplained)

* `MD5`, `SHA256` などのハッシュ値の生成ロジックを流れるように記述できる。
* JDK のライブラリを使うと、文字列からバイト配列にしてバイト配列から文字列にしてとか、煩雑な実装になってしまうが、これを使うとメソッドチェーンですっきり書ける。
* 任意のオブジェクトの変換も実装可能。

```java
String md5 = Hashing.md5().newHasher()
	.putLong(1L)
	.putString("あああ", Charsets.UTF_8)
	.putObject(employee, (from, into) -> into.putInt(from.getNo()).putString(from.getName(), StandardCharsets.UTF_8))
	.hash()
	.toString();	// 68676af35e77bff5d7500b9979ad5957
```

## Stopwatch

* パフォーマンスを計測するときに便利。
* 時間の単位を指定して計測値を取得可能。

```java
Stopwatch stopwatch = Stopwatch.createStarted();
IntStream.range(0, 1000).parallel().forEach(i -> {
	try {
		TimeUnit.MILLISECONDS.sleep(i);
	} catch (InterruptedException e) {
		e.printStackTrace();
	}
});
System.out.println(stopwatch.stop().elapsed(TimeUnit.MILLISECONDS) + "[msec]");
```

## [Splitter](https://github.com/google/guava/wiki/StringsExplained#splitter)

* 文字列を特定のパターンで分割する。
* 正規表現を使った分割を行うこともできるが、`CharMatcher` や文字列を使った分割が可能なので、単純な文字列マッチの同じパターンで多用する場合は、`String.split()` を毎回使うより高速に動作することが期待される。
* スレッドセーフで immutable なので、よく使うパターンは `static final` で持つことがオススメ。
* また、キーバリュー的な文字列を分割する `Splitter.MapSplitter` というのもある。
* ただし、`Splitter.MapSplitter` はキーの重複を許さないので、パラメータ名の重複を許す HTTP のクエリ文字列をパースしたいというような要件には実は合わなくて、何度かがっかりしている。

```java
// 全角スペースも分割対象
Splitter.on(CharMatcher.whitespace()).split("a bcd　e");	// [a, bcd, e]

Splitter.on("&").withKeyValueSeparator("=").split("key1=value1&key2=value2");	// {key1=value1, key2=value2}
Splitter.on("&").withKeyValueSeparator("=").split("key1=value1&key1=value2");	// キーが重複してしまうので IllegalArgumentException 発生
```

## まとめ

ということで、Guava の一部を列挙してみました。

Java をそのまま使うにはちょっと不便なところも強力に補完してくれますし、トレンドを反映した実装になっているので、非常に心地が良いのも特徴です。

ただ、採用した場合は、[`@Beta`](https://github.com/google/guava/wiki/PhilosophyExplained#beta-apis) なクラスもあったり、[`@Deprecated` になって消えていくこと](https://github.com/google/guava/wiki/PhilosophyExplained#non-beta-apis)もあるので、オープンソースを採用する以上必要なことですが、ライブラリのバージョンアップとともにプロダクトも追従していくことが不可欠です。

