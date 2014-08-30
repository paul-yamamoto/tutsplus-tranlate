＃SOLIDの原則: Part1 - 単一責任の原則(Single Responsibility Principle)

original:http://code.tutsplus.com/tutorials/solid-part-1-the-single-responsibility-principle--net-36074

単一責任(Single Responsibility), 拡張に対しては開き、修正に対しては閉じる(Open/Close),リスコフ置き換え(Liskov's Substitution, Interface Segregation),インターフェイス分離(Interface Segregation),依存性注入(Dependency Inversion)、これらのアジャイルソフトウェア開発の原則はどんな時でもコードを書く手助けになります。

##定義

**クラスに変更が起こる理由は、一つであるべき。(A class should have only one reason to change.)**

Rober C.Martin が彼の著書 Agile Softwear Development, Principles, Patterns, and Practcies で発表したこの原則はSOLIDの原則の1つです。「クラスに変更が起こる理由は、一つであるべき。」この原則の表しているいることはとてもシンプルですが、達成するのが難しい事です。

しかし何故、これがそれほど重要なのでしょうか？

静的なコンパイルが必要な言語では、複数の理由で変更が必要になれば、複数回の再デプロイが必要になります。クラスに対して2つの違う理由で変更が必要になれば、場合によっては、2つのチームが1つのソースを修正しなければならなくなります。そしてそれをそれぞれデプロイ、コンパイルした時に、片方の変更によりもう片方のチームのアプリケーションに組み込むことが出来なくなる事が起こり得ます。

例えコンパイルが不用な言語でも、一つのモジュールを二つの理由で変更したら、2つ目の修正の時に、1つ目の修正の再帰テストが必要です。これはQAチームの仕事の時間を奪い、手間を増やすことになります。



