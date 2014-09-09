#SOLIDの原則: Part1 - 単一責任の原則(Single Responsibility Principle)

original:http://code.tutsplus.com/tutorials/solid-part-1-the-single-responsibility-principle--net-36074

単一責任(Single Responsibility), 拡張に対しては開き、修正に対しては閉じる(Open/Close),リスコフ置き換え(Liskov's Substitution, Interface Segregation),インターフェイス分離(Interface Segregation),依存性注入(Dependency Inversion)、これらのアジャイルソフトウェア開発の原則はどんな時でもコードを書く手助けになります。

##定義

**クラスに変更が起こる理由は、一つであるべき。(A class should have only one reason to change.)**

Rober C.Martin が彼の著書 Agile Softwear Development, Principles, Patterns, and Practcies で発表したこの原則はSOLIDの原則の1つです。「クラスに変更が起こる理由は、一つであるべき。」この原則の表していることはとてもシンプルですが、達成するのが難しい事です。

しかし何故、これがそんなに重要なのでしょうか？

静的なコンパイルが必要な言語では、複数の理由で変更が必要になれば、複数回の再デプロイが必要になります。クラスに対して2つの違う理由で変更が必要になれば、場合によっては、2つのチームが1つのソースを修正しなければならなくなります。そしてそれをそれぞれデプロイ、コンパイルした時に、片方の変更によりもう片方のチームのアプリケーションに組み込むことが出来なくなる事が起こり得ます。

例えコンパイルが不用な言語でも、一つのモジュールを二つの理由で変更したら、2つ目の修正の時に、1つ目の修正の再帰テストが必要です。これはQAチームの仕事の時間を奪い、手間を増やすことになります。

##アプリケーション利用者

あるクラスやモジュールが単一責任の原則に沿っているかを見極めるのは、チェックリストを使ってチェック出来る様な単純な作業ではありません。そのクラスの変更理由を見つけ出す1つの方法として、そのクラスを使っている利用者を全て解析する方法があります。自分達の作ったクラスをモジュールとして組み込んだアプリケーションの利用者は、そのクラスを変更したい理由を持っている人です。下にモジュールと利用者の一覧を幾つか出してみます。

- **永続化モジュール** DBAやソフトウェアアーキテクトが利用者として挙げられます。
- **レポート生成モジュール** 販売員、会計などが利用者として挙げられます。
- **給与計算の支払い計算モジュール** 弁護士、管理職、会計担当などが利用者として挙げられます。
- **図書館管理システムの書籍検索モジュール** 司書と図書館の利用者が、利用者として挙げられます。

##役割とアクター（対象者）

一人の人物の全ての役割を列挙するのは大変難しい事です。小さな会社になれば大企業では別々の人物に割り当てられる役割りを一人で担わなければいけません。しかし、「役割り」そのものを定義するのは非常に難しい事です。「役割り」とは何でしょうか？私達はどのように「役割り」を見つけ出すのでしょうか？その方法として、対象者と彼等の「利用者」から「役割り」を見つけ出すのが簡単な方法です。

「利用者」が「対象者」に対して「変更する理由」を見出した時は「対象者」が「利用者」を見出したとも言えます。この方法で「対象者」から「役割り」を絞り込む事が出来ます。


##変更理由

上記の理由から、変更理由を見つけることがアクターを定義することとも言えます。

> 責務とは一つの「アクター」を定義する機能の集合である。 

> 責務に対する「アクター」とはその責務内の単一の変更理由と同義である。
> (Robert C Martin)

##既存コードの例

###自分自身を"Print"する機能を持っているオブジェクト
まずは`Book`クラスについて見て行きましょう。本の概要とそれに関する機能を持ったクラスです。
```
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function printCurrentPage() {
        echo "current page content";
    }
}
```

非常に理にかなったクラスの様に見えます。書籍が有りタイトル・著者名を保持していて、ページを送る機能もあります。
そして現在のページをプリントする機能も持っています。しかしこのクラスは少しばかり問題をはらんでいます。この`Book`クラスを使うアクターとして、どのような人物が考えられますか？とりあえず想像できるアクターとして、（司書が使うような）書籍の管理システムと、（利用者が文字のみ・もしくは画像としてページを取得するための）コンテンツのsy津力システムの２つが考えられます。そしてそれらは全く別物のアクターです。

プレゼンテーション処理にビジネスロジックを入れることは単一責任の原則(SRP)に反する、良くないプログラミングです。下の例を見てください。
```
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
interface Printer {
 
    function printPage($page);
}
 
class PlainTextPrinter implements Printer {
 
    function printPage($page) {
        echo $page;
    }
 
}
 
class HtmlPrinter implements Printer {
 
    function printPage($page) {
        echo '<div style="single-page">' . $page . '</div>';
    }
 
}
```

この例は単一責任の原則を守ってプレゼンテーションとビジネスロジックを分離した例です。こちらのほうがはるかに設計に拡張性を与えます。

###自分自身を"Save"する機能を持っているオブジェクト

似た様な例題としていプリントして保存する機能を持ったオブジェクトの例を見ます。

```
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function save() {
        $filename = '/documents/'. $this->getTitle(). ' - ' . $this->getAuthor();
        file_put_contents($filename, serialize($this));
    }
 
}
```

もう一度アクターを見直しましょう。保存処理に変更が必要になった場合このクラスの変更が必要になります。次のページを取得する処理に変更が必要になった時もこのクラスの変更が必要になります。このクラスには２つの異なる方向性の変更理由が存在しています。

```
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class SimpleFilePersistence {
 
    function save(Book $book) {
        $filename = '/documents/' . $book->getTitle() . ' - ' . $book->getAuthor();
        file_put_contents($filename, serialize($book));
    }
 
}
```
永続化処理を別クラスに分けることにより、責務をより明確に分かれ、永続処理に修正が必要になっても、`Book`クラスに対する修正は不要になりました。`DatabasePersistense`クラスを作ったとしても`Book`クラスとそれに関するビジネスロジックには何の影響も与えません。


##より高いレベルでの視点

私がこれまでこの記事で言及してきたのは、下の図に有るような一段高い視点での設計です。

この図で"単一責任の原則"が尊重されていることが分かりますか？この図の右側にはオブジェクトの生成を担うFactoriesと、プログラムのエントリポイントであるMianが配備されています。図の下側にはPersistence処理が配備されています。モジュールとして分離されている物は、それぞれ別の責務を負っています。この図の左側にはPresentation処理とDelivery処理が置かれています。MVCなどのUIがここに当たるでしょう。"単一責任の原則"を再度尊重しましょう。これらを組み合わせて私達はビジネスロジックを構築していくのです。

##ソフトウェア設計を考慮すると

普段書いているソフトウェアについて考えると、色々な視点での分析が可能な事に気づくと思います。同じクラスに対して何度もくる要求は、そのクラスの変更理由の方向性を表していると言えます。この変更理由の方向性は、単一責任の原則を適用するための手掛かりになります。同じ機能のまとまりに対して、グループ化出来る要求があれば、その機能の変更理由を特定出来る可能性が高まります。

ソフトウェアの一番重要な価値は変更が容易な事です。２番めに重要な価値は機能性です。ユーザーの要求を可能な限り満たす事です。第２の価値を最大限高めるためには、第１の価値が必須になります。第一の価値を高い状態で維持するには、拡張性や変更性を設計で考慮する必要があります。それは単一責任の原則を遵守する事と同じです。

何故そうなるのか順に説明すると
1. 第一の価値が高いソフトウェアは第２の価値を素早く提供できます。
1. 第２の価値は言い換えればユーザーのニーズです。
1. ユーザーのニーズはアクターのニーズ（関係者）と言い換える事が出来ます。
1. アクターのニーズは、アクターによる変更理由と言い換える事が出来ます。
1. アクターによる変更理由は、我々の責務を定義するものです。

つまり、私達がソフトウェアを設計する時に必要なのは以下の作業です。
1. アクターを見つけて定義する。
1. アクターによって導き出される責務を見つける。
1. 一つの責務を負うようにクラスやファンクションをグループ化する。

##少し分かりにくい例

```
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
    function getLocation() {
        // returns the position in the library
        // ie. shelf number & room number
    }
 
}
```

このクラスはとてもリーズナブルに見えますよね。永続化や表示の処理はこのクラスに含まれていません。
このクラスには`turnPage()`とその他の書籍の情報を提供するメソッドが含まれています。しかしこのクラスは問題を含んでいます。このクラスをもう一度検討して見てください。`getLocation()`このメソッドは問題になり得ます。

`Book`クラスの全てのメソッドはビジネスロジックです。このクラスの設計もビジネスで使われる視点で考えなければいけません。もしもこのクラスが実際の図書館の司書が書籍の検索・貸出のために使われているとしたら、単一責任の原則が破られていると言えます。

`getTitle()`, `etAuthor()`,`getLocation()`これらのメソッドがこのクラスにあるのは理由があります。このクラスを利用するアプリケーションでは本を検査して、その本の最初のページを表示してその本が探していたものかどうか手助けする事でしょう。この時アプリケーションは`getLocation()`以外のメソッドを全て使います。そして図書館の利用者が書籍の図書館での場所を気にする事は普通はありません。司書が探し出して渡してくれるからです。このクラスを単一責任の原則に沿うように修正してみましょう。

```
class Book {
 
    function getTitle() {
        return "A Great Book";
    }
 
    function getAuthor() {
        return "John Doe";
    }
 
    function turnPage() {
        // pointer to next page
    }
 
    function getCurrentPage() {
        return "current page content";
    }
 
}
 
class BookLocator {
 
    function locate(Book $book) {
        // returns the position in the library
        // ie. shelf number & room number
        $libraryMap->findBookBy($book->getTitle(), $book->getAuthor());
    }
 
}
```

今回新しく`BookLocator`が追加されました。図書館の司書のためのクラスです。図書管理利用者が使うのは`Book`クラスのみです。もちろん`BookLocator`には複数の実装方法があります。タイトル・作者その他`Book`オブジェクトの各種情報を利用するでしょう。しかしそれは仕様次第です。重要なのは書架に変更があっても、司書が別の図書館の情報を扱うようになったとしても、`Book`クラスには何の影響も与えないという事です。逆の事を想定しても同様で、書籍のページ表示システムが最初のページを表示するものから本の要約を表示する物になっても、司書向けのシステム・書籍の検索処理に何の影響も与えないという事です。

もしも図書館向けシステムがセルフサービスを前提としたものであれば、最初の例でも単一責任の原則を守っていると言えます。図書館利用者も司書も本を探し出す必要があるからです。重要なのは自分達のユーザー・ビジネスを念頭に置いて設計する事です。