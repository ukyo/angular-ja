# Hierarchical Dependency Injectors

Angularの依存性の注入システムは_階層的_です。
アプリのコンポーネントツリーと並行する形でインジェクターツリーを持ちます。
そのコンポーネントツリーの任意の階層でインジェクターを再設定できます。

このガイドはこのシステムとあなたの利点のためにそれをどのように使うかを探ります。
ここでは、この<live-example></live-example>をベースにした例を使用しています。

{@a ngmodule-vs-comp}
{@a where-to-register}

## Where to configure providers

インジェクター階層内のさまざまなインジェクターにプロバイダーを設定できます。
内部プラットフォームレベルのインジェクタは、実行中のすべてのアプリで共有されています。
`AppModule`インジェクタはアプリ全体のインジェクタ階層のルートです。
NgModuleでは、ディレクティブレベルのインジェクタはコンポーネント階層の構造に従います。

プロバイダを設定する場所を選択すると、最終的なバンドルサイズ、サービスの_スコープ_、およびサービスの_ライフタイム_が異なります。

サービス自体の(通常はアプリのルートレベルで)`@Injectable()`デコレータでプロバイダを指定すると、CLIのプロダクションビルドで使用されているような最適化ツールは*ツリーシェイキング*を実行できます。これによって、アプリで使用されていないサービスが削除されます。ツリーシェイキングの結果、バンドルサイズがより小さくなります。

* [ツリーシェイキング可能なプロバイダー](guide/dependency-injection-providers#tree-shakable-providers)について詳しく学んでください。

あなたはアプリの至る所で `UserService`を注入する可能性があり、毎回同じサービスインスタンスを注入したいと思うでしょう。`root`インジェクタを通して`UserService`を提供することは良い選択です、そしてアプリでサービスを生成するときに[Angular CLI](cli)はデフォルトで使用します。

<div class="alert is-helpful">
<header>プラットフォームインジェクター</header>

`providedIn: 'root'`を使うとき、_アプリ_のルートインジェクタを設定しています。これは、`AppModule`のインジェクタです。
インジェクタ階層全体の実際のルートは、アプリのルートインジェクタの親である_プラットフォームインジェクター_です。
これにより、複数のアプリがプラットフォームの設定を共有できます。たとえば、ブラウザには1つのURLバーしかありません。実行しているアプリの数にかかわらずです。

プラットフォームインジェクターは、ブートストラップ中に内部的に使用され、プラットフォーム固有の依存関係を設定します。`platformBrowser()`関数を使って `extraProviders`を提供することでプラットフォームレベルで追加のプラットフォーム固有のプロバイダを設定できます。

インジェクタ階層による依存関係の解決についてさらに学びましょう。
[What you always wanted to know about Angular Dependency Injection tree](https://blog.angularindepth.com/angular-dependency-injection-and-tree-shakeable-tokens-4588a8f70d5d)


</div>

*NgModuleレベル*のプロバイダーは`@NgModule()`の`providers`メタデータオプションで、あるいは`@Injectable()`の`providedIn`オプションで（ルートの`AppModule`以外のモジュールで）指定できます。

モジュールが[遅延ロード](guide/lazy-loading-ngmodules)されている場合、`@ NgModule()`の`providers`オプションを使用してください。そのモジュールがロードされると、モジュール自身のインジェクタはプロバイダに設定され、Angularはそのモジュール内に作成する任意のクラスに対応するサービスを注入できます。`@Injectable()`オプションで`providedIn: MyLazyloadModule`を使用する場合、プロバイダがアプリケーション内の他の場所で使用されていなければ、プロバイダはコンパイル時にツリーシェイクできます。

* [ツリーシェイキング可能なプロバイダー](guide/dependency-injection-providers#tree-shakable-providers)について詳しく学んでください。

ルートレベルとモジュールレベルの両方のインジェクタの場合、サービスインスタンスはアプリケーションまたはモジュールの存続期間中存続し、Angularはそれを必要とするすべてのクラスにこの1つのサービスインスタンスを注入します。

*コンポーネントレベル*のプロバイダは、各コンポーネントインスタンスのインジェクタを設定します。
Angularは、対応するサービスをそのコンポーネントインスタンスまたはその下位コンポーネントインスタンスの1つにのみ注入できます。
Angularは他の場所に同じサービスインスタンスを注入することはできません。

コンポーネントから提供されるサービスは有効期間が限られている場合があります。
それぞれのコンポーネントの新しいインスタンスは、自身のサービスインスタンスを取得します。
コンポーネントインスタンスが破棄されると、そのサービスインスタンスも破棄されます。

私たちのサンプルアプリでは、アプリケーションの起動時に`HeroComponent`が作成され、
そしてそれは決して破棄されないので、`HeroComponent`用に作成された` HeroService`インスタンスはアプリの存続期間中存続します。
`HeroService`アクセスを`HeroComponent`とそのネストされた`HeroListComponent`に制限したい場合、
`HeroListComponent`は、`HeroComponent`メタデータの中で、
コンポーネントレベルで`HeroService`を提供してください。

* 詳細は、次の[コンポーネントレベルの依存性の注入の例](#component-injectors)を参照してください。


{@a register-providers-injectable}

### @Injectableレベルの設定 

`@Injectable()`デコレータはすべてのサービスクラスを識別します。
サービスクラスの`providedIn`メタデータオプションは装飾されたクラスをサービスの提供者として使用するために特定のインジェクタを設定します(通常は` root`)。
注入可能なクラスが独自のサービスを`root`インジェクタに提供するとき、そのサービスはクラスがインポートされた場所ならどこでも利用可能です。

次の例では、クラスの`@Injectable()`デコレータを使って`HeroService`のプロバイダを設定しています。

<code-example path="dependency-injection/src/app/heroes/hero.service.0.ts"  header="src/app/heroes/heroes.service.ts" linenums="false"> </code-example> 

この設定はAngularに、アプリケーションのルートインジェクタが
コンストラクタを呼び出して`HeroService`のインスタンス、
そして、そのインスタンスをアプリケーション全体で利用できるようにします。

アプリのルートインジェクタでサービスを提供するのが典型的なケースです。
そしてCLIは新しいサービスを生成するとき、
あなたに代わってこの種のプロバイダを自動的に設定します。
ただし、サービスを常にルートレベルで提供したくない場合があります。
たとえば、ユーザーにサービスの使用を明示的にオプトインさせたい場合があります。

`root`インジェクタを指定する代わりに、`providedIn`を特定のNgModuleに設定することができます。

例えば、次の抜粋では、`HeroModule`を含むすべてのインジェクタで利用可能なように`@Injectable()`デコレータはプロバイダを設定しています。

<code-example path="dependency-injection/src/app/heroes/hero.service.4.ts"  header="src/app/heroes/hero.service.ts" linenums="false"> </code-example>

これは一般にNgModule自身のインジェクタを設定することと同じです。
ただし、
NgModuleがそれを使用しない場合、
サービスはツリーシェイク可能です。
一部のコンポーネントがオプションで注入したい*かもしれない*特定のサービスを提供し、
そのサービスを提供するかどうかをアプリに任せることができるライブラリに役立ちます。

* [ツリーシェイキング可能なプロバイダー](guide/dependency-injection-providers#tree-shakable-providers)について詳しく学んでください。


### @NgModuleレベルのインジェクター

プロバイダのスコープをそのモジュールに限定するために、非ルートのNgModuleのために `providedIn`メタデータオプションを使ってモジュールレベルでプロバイダを設定することができます。
これは `@Injectable()`メタデータでルート以外のモジュールを指定するのと同じですが、この方法で提供されるサービスはツリーシェイクできない点が異なります。

アプリの`root`インジェクタは`AppModule`インジェクタなので、通常`providedIn`に`AppModule`を指定する必要はありません。
ただし、`AppModule`の`@NgModule()`メタデータでアプリ全体のプロバイダを設定すると、
`@Injectable()`メタデータの `root`用に設定されたものを上書きします。
これを実行して、複数のアプリで共有されるサービスのデフォルト以外のプロバイダを設定できます。

コンポーネントのルーター設定に、
`AppModule`の`providers`リストに非デフォルトの
[ロケーションストラテジー](guide/router#location-strategy)を追加する場合の例は次のようになります。

<code-example path="dependency-injection-in-action/src/app/app.module.ts" region="providers" header="src/app/app.module.ts (providers)" linenums="false">

</code-example>


{@a register-providers-component}

### @Componentレベルのインジェクター

NgModule内の個々のコンポーネントには自身のインジェクターを持ちます。
`@Component`メタデータを使ってコンポーネントレベルでプロバイダーを設定することによって、
プロバイダーの範囲をコンポーネントとその子に制限できます。

次の例は、`providers`配列で`HeroService`を指定するように`HeroesComponent`を修正したものです。`HeroService`はこのコンポーネントのインスタンス、または任意の子コンポーネントのインスタンスにヒーローたちを提供することができます。

<code-example path="dependency-injection/src/app/heroes/heroes.component.1.ts" header="src/app/heroes/heroes.component.ts" linenums="false">
</code-example>

### 要素インジェクター

インジェクターは実際にはコンポーネントに属しているのではなく、DOM内のコンポーネントインスタンスのアンカー要素に属しています 異なるDOM要素の異なるコンポーネントインスタンスは、異なるインジェクターを使用します。

コンポーネントは特別な種類のディレクティブであり、
`@Component`の`providers`プロパティは`@Directive()`から継承されます。
それらの`@Directive()`メタデータに、
ディレクティブも依存関係を持つことができ、プロバイダーを設定できます
`providers`プロパティを使ってコンポーネントやディレクティブのプロバイダを設定するとき、そのプロバイダーはアンカーDOM要素のインジェクターに属します。同じ要素のコンポーネントとディレクティブはインジェクターを共有します。

<!--- TBD with examples
* For an example of how this works, see [Element-level providers](guide/dependency-injection-in-action#directive-level-providers).
--->

* [Angularの要素インジェクター](https://blog.angularindepth.com/a-curios-case-of-the-host-decorator-and-element-injectors-in-angular-582562abcf0a)についてさらに学びましょう。



## インジェクターバブリング

このガイドのツアー・オブ・ヒーローズアプリケーションに関するバリエーションについて考察してみましょう。
一番上にあるのは`HeroesListComponent`のようないくつかのサブコンポーネントを持つ`AppComponent`です。
`HeroesListComponent`は`HeroTaxReturnComponent`の複数のインスタンスを保持し管理します。
次の図は3つの`HeroTaxReturnComponent`インスタンスが同時に開いているときのこの3階層のコンポーネントツリーの状態を表しています。

<figure>
  <img src="generated/images/guide/dependency-injection/component-hierarchy.png" alt="injector tree">
</figure>

コンポーネントが依存関係を要求すると、Angularはそのコンポーネント自身のインジェクターに登録されているプロバイダーでその依存関係を満たそうとします。
コンポーネントのインジェクターにプロバイダーがない場合は、リクエストをその親コンポーネントのインジェクターに渡します。
そのインジェクターが要求を満たすことができない場合は、ツリーの上の次の親インジェクターに要求を渡します。
Angularがリクエストを処理できるインジェクターを見つけるまで、または先祖インジェクターがなくなるまで、リクエストはバブリングを続けます。
先祖がなくなると、Angularはエラーをスローします。

異なるレベルで同じDIトークンのプロバイダを登録した場合、Angularが最初に遭遇するのが依存関係を提供するために使用するプロバイダです。たとえば、プロバイダーがサービスを必要とするコンポーネントにローカルに登録されている場合、Angularは同じサービスの別のプロバイダーを探しません。


<div class="alert is-helpful">

コンポーネントのコンストラクタ内の依存するサービスのパラメータに
`@Host()`パラメータデコレーターを追加することでバブリングを抑えることができます。
プロバイダーの探索は、コンポーネントのホスト要素のインジェクターで停止します。

* `@Host`とともに、プロバイダが見つからない場合にnullケースを処理できるようにする別のパラメータデコレータの`@Optional`を使用している[例](guide/dependency-injection-in-action#qualify-dependency-lookup)を参照してください。

* [`@Host`デコレーターと要素インジェクター](https://blog.angularindepth.com/a-curios-case-of-the-host-decorator-and-element-injectors-in-angular-582562abcf0a)についてさらに学びましょう。

</div>

プロバイダを最上位(通常はルートのAppModule)でルートインジェクターに登録するだけでは、インジェクターのツリーはフラットに見えます。
`bootstrapModule`メソッドで設定したか、自身のサービスで全てのプロバイダーを` root`で登録したかに関わらず、全てのリクエストはルートインジェクターにバブルアップします。

{@a component-injectors}

## コンポーネントインジェクター

さまざまなレベルで1つ以上のプロバイダを設定する機能により、興味深く有用な可能性が生まれます。
ガイドサンプルでは、必要に応じていくつかのシナリオを説明します。

### シナリオ: サービスの隔離

アーキテクチャ上の理由から、サービスへのアクセスをそのサービスが属するアプリケーションドメインに制限する可能性があります。
例えば、ガイドサンプルは悪役のリストを表示する`VillainsListComponent`を含みます。
それはそれらの悪役を`VillainsService`から取得します。

`VillainsService`をルートの`AppModule`(あなたが`HeroesService`を登録した場所)に指定した場合、
これは_Hero_ワークフローを含めて`VillainsService`をアプリケーションのいたるところで利用可能にします。後で `VillainsService`を修正した場合、どこかのヒーローコンポーネントの中で何かを壊すことができます。ルートのAppModuleにサービスを提供することはその危険性を生み出します。

代わりに、`VillainsListComponent`の`providers`メタデータで `VillainsService`を次のように提供することができます:


<code-example path="hierarchical-dependency-injection/src/app/villains-list.component.ts" linenums="false" header="src/app/villains-list.component.ts (metadata)" region="metadata">

</code-example>

`VillainsListComponent`メタデータで`VillainsService`を提供し、それ以外の場所では提供しません。
このサービスは`VillainsListComponent`とそのサブコンポーネントツリーでのみ利用可能になります。
まだシングルトンですが、_villain_ドメインにのみ存在するシングルトンです。

これで、主人公のコンポーネントがアクセスできないことがわかりました。 あなたはエラーへの露出を減らしました。

### シナリオ: 複数の編集セッション

多くのアプリケーションでは、ユーザーは複数の未解決タスクを同時に処理できます。
たとえば、納税申告アプリケーションでは、作成者は複数の納税申告書を作成し、
1日を通して納税申告書を切り替えることができます。

このガイドでは、そのシナリオをツアー・オブ・ヒーローズのテーマの例で説明します。
スーパーヒーローのリストを表示する外側の`HeroListComponent`を想像してください。

主人公の確定申告を開くには、作成者は主人公の名前をクリックします。これにより、その申告を編集するためのコンポーネントが開きます。
選択された各ヒーロー納税申告書はそれぞれの構成要素で開き、複数の申告書を同時に開くことができます。

各納税申告コンポーネントには以下の特性があります。

* 独自の納税申告編集会です。
* 他の構成要素での申告に影響を与えずに確定申告を変更することができます。
* その納税申告書への変更を保存したり、それらをキャンセルすることができます。


<figure>
  <img src="generated/images/guide/dependency-injection/hid-heroes-anim.gif" alt="Heroes in action">
</figure>

`HeroTaxReturnComponent`に変更を管理し復元するロジックがあるとします。
それは単純なヒーローの納税申告書のためのかなり簡単な仕事でしょう。
現実の世界では、豊富な納税申告データモデルを使用しているため、変更管理には注意が必要です。
この例のように、その管理をヘルパーサービスに委任することができます。

これが`HeroTaxReturnService`です。
それは単一の`HeroTaxReturn`をキャッシュし、その戻りに対する変更を監視し、それを保存または復元することができます。
また、インジェクションによって取得されるアプリケーション全体のシングルトン `HeroService`に委任します。


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.service.ts" header="src/app/hero-tax-return.service.ts">

</code-example>

これを使用している`HeroTaxReturnComponent`は次のようになります。


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.component.ts" header="src/app/hero-tax-return.component.ts">

</code-example>


_tax-return-to-edit_は、ゲッターとセッターで実装されているinputプロパティを介して到着します。
セッターはコンポーネント自身の`HeroTaxReturnService`のインスタンスを入ってくるリターンで初期化します。
ゲッターは、そのサービスが言うことが常に主人公の現在の状態であることを返します。
コンポーネントはまた、この納税申告書を保存および復元するようにサービスに要求します。

サービスがアプリケーション全体のシングルトンの場合、これは機能しません。
各コンポーネントは同じサービスインスタンスを共有し、各コンポーネントは別のヒーローに属する納税申告書を上書きします。

これを防ぐために、コンポーネントメタデータの`providers`プロパティを使って、サービスを提供するように`HeroTaxReturnComponent`のコンポーネントレベルのインジェクターを設定します。


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.component.ts" linenums="false" header="src/app/hero-tax-return.component.ts (providers)" region="providers">

</code-example>

`HeroTaxReturnComponent`は`HeroTaxReturnService`の独自のプロバイダーを持っています。
すべてのコンポーネント_インスタンス_には独自のインジェクタがあります。
コンポーネントレベルでサービスを提供することで、コンポーネントのすべてのインスタンスが独自のサービスのプライベートインスタンスを取得し、確定申告が上書きされることがなくなります。


<div class="alert is-helpful">


シナリオコードの残りの部分は、ドキュメントの他の部分で学ぶことができる他のAngularの機能とテクニックに依存しています。
あなたはそれを確認し、<live-example></live-example>からそれをダウンロードすることができます。


</div>



### シナリオ: 特別なプロバイダー

別のレベルでサービスを再提供するもう1つの理由は、そのサービスのより専門的な実装をコンポーネントツリーのより深いところに置き換えることです。

いくつかのサービスに依存するCarコンポーネントを考えます。
ルートインジェクター(Aとマークされている)に`CarService`、
`EngineService`、および `TiresService`という_ジェネリック_プロバイダーを設定したとします。

これら3つの汎用サービスから構築された自動車を表示する自動車コンポーネント(A)を作成します。

それから `CarService`と` EngineService`のためのそれ自身の_特別な_プロバイダーを定義する子コンポーネント(B)を作成します
それはコンポーネント(B)で起こっていることすべてに適した特別な能力を持っています。

コンポーネント(B)は`CarService`のためのそれ自身の、さらに専門化されたプロバイダーさえも定義する別のコンポーネント(C)の親です。


<figure>
  <img src="generated/images/guide/dependency-injection/car-components.png" alt="car components">
</figure>

舞台裏では、各コンポーネントは、そのコンポーネント自体に対して定義された0、1、または複数のプロバイダーを使用して各自のインジェクタを設定します。

最も深い部分(C)で`Car`のインスタンスを解決するとき、
そのインジェクターは`Engine`はインジェクター(B)によって解決され、
そして`Tires`はルートインジェクター(A)によって解決され、インジェクター(C)によって解決された`Car'のインスタンスを生成します。



<figure>
  <img src="generated/images/guide/dependency-injection/injector-tree.png" alt="car injector tree">
</figure>



<div class="alert is-helpful">


この_cars_シナリオのコードはサンプルの`car.components.ts`と` car.services.ts`ファイルにあります。
<live-example></live-example>から確認してダウンロードできます。

</div>
