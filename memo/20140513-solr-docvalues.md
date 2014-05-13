# Solr の DocValues

最近、仕事で Solr をいじっています。

## 文字列のソートで OutOfMemoryError

担当のデータが1億件もあってフリガナで辞書順に sort したいのだけど、単純に `indexed="true"` にしてソートさせたら、OutOfMemoryError で完全に動作しない。
物理サーバは 16GB あって、JavaVM に 8GB 与えているので、それなりにリソースはあるはずなのに。

多分、1億件の string の index がメモリを食いつぶしていて、実際 Solr を立ち上げた直後で `ps aux` の `RSS` の値 (実メモリ使用量) が 8GB に達しているので、どうにかして index サイズを小さくしないとダメなんではないかと試行錯誤してみた。

## 数値型への変換

基本 Solr の sort に string などのテキスト型を使わないのがセオリーみたいで、ところどころで *numeric type にしろ* みたいな記載がある。
じゃ、フリガナを辞書順にするような数値に変換すればいいのかもしれないが、

* Solr にデータを indexing する前に1億件をソートして置かなければならない。
  * 日単位の indexing 時間を考慮しないといけない。
* ソートしないで indexing するとなると、フリガナを辞書順に並べる数字をフリガナから算出する必要がある。
  * この場合、「あ」→「01」、「い」→「02」みたいにすると、string で sort するのとあまり変わらないので労力かかるだけ。
  * それに、long とかで収まりきらない桁数になってしまう。

ということで他の方法を考えることに。

## disk を使う docValues

そんな折に偶然[こんなページ](https://cwiki.apache.org/confluence/display/solr/DocValues)を見つける。

騙されたと思って↓の設定を追加してみて、indexing してみた。

* schema.xml
  * `<field name="ruby" type="string_ondisk" indexed="false" stored="false" docvalues="true"></field>`
  * `<fieldtype name="string_ondisk" class="solr.StrField" docvaluesformat="Disk"></fieldtype>`
* solrconfig.xml
  * `<codecfactory class="solr.SchemaCodecFactory"></codecfactory>`

これでフリガナで sort しても OutOfMemoryError が発生しなくなった。

DocValues は sort や facet などでは通常の転置インデックスより効率的らしい。
さらに、docValuesFormat で Disk を指定しているので、メモリを使わずに sort できているらしい。indexed="false" でも sort に使えました。

あまり利用例が見つからなかったので、あまり使っている人はいないのかもしれない。
けど、とりあえずうまく行っているので、これで様子を見ることにする。
