[Laravel](http://laravel.com/)を使って開発をしている人は数多くいますが、フレームワークの機能にまで手を入れる人はごくわずかでしょう。実際Laravelのドキュメントはほとんどの場面を網羅しています。しかし、すべてを網羅している訳ではありません。

誤解してほしくないのですが、Laravelのでキュメントは素晴らしいもので、必要なことは大抵記載してあります。しかし、「全て」を記載するということは難しいことです。
これから、その記載されていない隠れた機能について紹介します。

## Cascading Views

利用可能: v4.0.0以降

ドキュメント: なし

LaravelのViewはconfigurationファイルと同じように、入れ子にする事ができます。Cascading viewsは拡張しやすいテーマシステムを導入するのに非常に便利です。
下がディレクトリ構成の例です。

```
/app
    /views/
	    /blog
			/inde.blade.php
/themes
	/default
		/views
			/blog/
				/index.blade.php
			/theme.blade.php
```
この構成で`return View::make('theme::blog.index');`とした時に、最初に`themes/defautl/views`にbladeファイルを探しに行き、見つからなければ`app/view`ディレクトリを探しに行くようになります。

このgemを使用するために`View::addNamespace`で２つのnamespaceを登録する必要があります。
```
View::addNamespace('theme', [
    base_path().'/themes/default/views',
    app_path().'/views'
]);
```

## Collections

利用可能: v4.0.0以降

ドキュメント: 一部あり

Colectionsは配列の操作・管理のための素晴らしいツールです。Collectionsには多くの手軽なメソッドと、`ArrayableInterface`,`IteratorAggregate`,`JsonableInterface`など便利なインターフェイスの実装が含まれています。

ブログシステムを構築する例で使い方を紹介しましょう。ブログの記事を表す`Ariticle`classのインスタンスで構成される配列を操作したい場合は、最初にそれを引数にして`Collection`クラスを作る必要があります。

```
$articles = new Illuminate\Support\Collection($arrayOfArticles);
```

### Sorting
`Collections`を使うと記事の並び替えが可能です。例えば記事一覧を最近更新された順に並び替えてみましょう。記事一覧がファイルシステムから読み込まれた時に、`Article`クラスの`updateAt`に最終更新日が設定される場合、並び替えは以下の様にします。

```
$articles->sortByDesc(function ($article) { 
    return $article->updatedAt; 
});
```
`sortBy`と`sortByDesc`メソッドは並び替えに必要な値を返すcallbackを引数にとり配列の並び替えを実行します。このケースでは単純に記事の最終更新日を返していて、`Collections`はそれを元に降順で並び替えをしています。

### Filtering

配列に対して並び替えだけでなく絞り込みをする事も可能です。MySQLで`WHERE`句を使うように簡単に出来ます。先程の記事一覧を絞り込んでみましょう。
```
<?php
 
$searchQuery = 'Laravel rocks!';
 
$results = $articles->filter(function ($article) use ($searchQuery) {
    return preg_match(sprintf('/%s/m', $searchQuery), $article->body);
});
```
`filter`メソッドは絞り込みを適用した新しい`Illuminate\Support\Collection`のインスタンスを返すので、`$results`にそれを代入する必要があります。新しい配列には本文に"Laravel rocks!"が含まれる記事のみが含まれる様になります。

### Pagination
`Collections`を使ってページ送りを実装する事も出来ます。1ページに全ての記事を載せる必要はありません。
```
$perPage = 1;
 
$page = Input::get('page', 1);
 
if ($page > ($articles->count() / $perPage)) {
    $page = 1;
}
 
$pageOffset = ($page * $perPage) - $perPage;
 
$results = $articles->slice($pageOffset, $perPage);
```
`slice`メソッドをを使って、必要な位置の記事一覧を切り出して、`$results`に代入しています。

この方法でLaravelの`Paginator`インスタンスを生成することも出来ます。全てのページのページ番号とリンクを作ることもが出来るのです。

### そしてさらに！
記事の順番をランダムにするのも簡単です。
```
$article = $articles->random();
```
配列のようにfor句でループさせる事も出来ます。`IteratorAggregate`と`ArrayIterator`インターフェイスを実装しているからです。
```
foreach ($articles as $article) {
    echo $article->body;
}
```
普通の配列やJSONに変換するのも簡単です。
```
$array = $articles->toArray();
$json = $articles->toJson();
```
数あるメソッドの中で最もcoolなものの一つが`groupBy`でしょう。一つのキーを元に配列をグループ化することが出来ます。

ブログ記事一覧の例で言えば、記事のプロパティのcategoryでグループ化するには以下のようにします。
```
$results = $articles->groupBy('category');
```
これで全ての記事はカテゴリ毎にグループ化されました。
更に特定のカテゴリの記事一覧を取得するには以下の様にします。
```
foreach ($results->get('tutorial') as $article) { 
    echo $article->body; 
}
```
`Collections`はLaravelが提供する隠れ機能の中で最も便利な物の1つです。

## Regular Expression Filters

利用可能: v4.1.19以降

ドキュメント: なし

ルートのフィルタリングはほとんどのプロジェクトで行う、よくある作業です。ルーティングの際にユーザー認証や年齢制限などをかける事も出来ます。`Router::filter`でフィルターを作成して、個別にルートティングを設定したり、ルーティングをグループ化して、`Router::when`を使ってパターンマッチに適用させて使用します。

例:
```
Route::filter('restricted', function($route, $request, $group)
{
    // Restrict user access based on the value of $group
});

Route::when('admin/*', 'restricted:admin');
```

上の例では最初に`restricted`を作成しています。フィルターには`$group`パラメーターを渡すように設定されています。( `$route`と`$request`はbefore filterにいつも渡されるパラメーターです。 )

しかし、より柔軟に適応したい場合はどうすれば良いでしょうか？例えばurl`admin`にはフィルターを適用したいけど、`admin/login`にはフィルターを適用したくない時です。一つの方法としてroute groupを作成して、ログインページだけそこから外す方法があります。そしてもう一つの方法として`Route::whenRegex`が使い正規表現で適用させる方法です。
```
Route::whenRegex('/^admin(\/(?!login)\S+)?$/', 'restricted:admin');
```
この正規表現は'`admin`には適用するけど、後ろに`/login`が付いているものには適用しないというパターンです。素晴らしい事にこれだけで`admin/login`以外の全てのadminページに適用する`restricted:admin`フィルターが出来ます。

## The Message Bag

利用可能: v4.0.0以降

ドキュメント: 一部あり

あなたが自覚していなくても、`Illuminate\Support\MessageBag` をどこかで使っているはずです。`MessageBag`の最大の役割はLaravelに組み込まれているValidatorを使った時に、そのエラーを保持することです。
**全てのviewに存在する`$errors`変数は空の`MessageBag`インスタンスか`Redirect::to('/')->withErrors($validator);`で、セッションにフラッシュしたインスタンスを保持しています。**

フォームでの入力エーの時に、正しい入力方法を表示する時などにこれは使えます。
```
{{ Form::text('username', null) }}
@if($errors->has('username'))
    <div class="error">{{ $errors->first('username') }}></div>;
@endif
```
上のテンプレートでは`if`句は必要ありません。`first`メソッドを使い２つ目の引数に`div`ブロックを渡せば、メッセージを簡単にwrapさせる事が出来ます。
```
{{ Form::text('username', null) }}
{{ $errors->first('username', '<div class="error">:message</div>') }}
```
テンプレートが本当に良い感じになりましたね！

### Fluent

利用可能: v3.0.0以降

ドキュメント: 一部あり

