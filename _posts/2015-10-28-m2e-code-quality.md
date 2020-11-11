---
tags: eclipse, Java, maven
---

# eclipse + maven + m2e-code-quality で Java コードの品質を保つ

## m2e-code-quality とは

[m2e-code-quality](http://m2e-code-quality.github.io/m2e-code-quality/) は、eclipse の m2e の拡張プラグインで、m2e 同様、pom.xml のコード品質に関わる設定を eclipse の設定に反映します。

これによって、初回の maven プロジェクトのインポートでチームメンバーに設定を強制することができるので、面倒な設定手順をいちいち共有しなくても良くなります。

## 対応するツール

m2e-code-quality に対応するツールは以下のとおり。

* FindBugs
    * コンパイル後のバイトコードを解析し、ソフトウェアの不具合を探してくれるツール
* PMD
    * Java のソースコードを解析し、ソフトウェアの不具合を探してくれるツール
* Checkstyle
    * コーディング規約をチェックするための静的解析ツール

pom.xml に上記の設定を書いておくと、eclipse のプラグインも動いてくれます。

Jenkins 上で maven に FindBugs, PMD, Checkstyle のレポートを実行をしている場合、その設定と同じことをローカルの eclipse 上でも実行できるので、ローカルと CI のコード品質チェックを一致させることができます。

## 設定方法

* プラグインが対応しているかどうかによって対応が異なりますが、maven の設定をしておくのが基本です。

### FindBugs

pom.xml には [findbugs-maven-plugin](http://gleclaire.github.io/findbugs-maven-plugin/) が実行できるような設定を追記しておきます。

```xml:pom.xml
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>findbugs-maven-plugin</artifactId>
	<executions>
		<execution>
			<id>fb</id>
			<goals>
				<goal>findbugs</goal>
			</goals>
			<phase>compile</phase>
		</execution>
	</executions>
	<configuration>
		<effort>Default</effort>
		<omitVisitors>[ディテクタ]</omitVisitors>
	</configuration>
</plugin>

```

* 注意点
    * visitors を指定すると、全部 OFF にして visitors にあるディテクターだけ ON になる。omitVisitors を指定すると、デフォルトの状態から指定のディテクターを OFF にするらしい。

### Checkstyle

pom.xml には [maven-checkstyle-plugin](https://maven.apache.org/plugins/maven-checkstyle-plugin/) が実行できるような設定を追記しておきます。

```xml:pom.xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-checkstyle-plugin</artifactId>
	<executions>
		<execution>
			<id>checkstyle</id>
			<phase>compile</phase>
			<goals>
				<goal>checkstyle</goal>
			</goals>
		</execution>
		<configuration>
			<configLocation>[URL or 相対パス]</configLocation>
		</configuration>
	</executions>
</plugin>
```

### PMD

省略

## まとめ

* コード品質を保つ設定を pom.xml にだけ書くことで eclipse に反映する方法をまとめました。
* プラグインの開発によって eclipse の反映がうまくいかないことがあるかもしれませんが、pom.xml にプロジェクト設定を一元化し、その設定を見て、IDEやその他の環境に反映されるという環境づくりは、各人の開発環境の独立性を確保し、好きな開発環境を構築することができます。eclipse でなくても netbeans などで pom.xml の設定を反映できれば、自由な開発環境をエンジニアが選択できます。また、Jenkins などの CI 環境での動作も保証できます。
* maven のプロジェクトのポータビリティ性を活用して、Java エンジニアの自由度を上げた開発環境を構築してみたはどうでしょうか？

