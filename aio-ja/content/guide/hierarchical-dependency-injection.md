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

Individual components within an NgModule have their own injectors.
You can limit the scope of a provider to a component and its children
by configuring the provider at the component level using the `@Component` metadata.

The following example is a revised `HeroesComponent` that specifies `HeroService` in its `providers` array. `HeroService` can provide heroes to instances of this component, or to any child component instances. 

<code-example path="dependency-injection/src/app/heroes/heroes.component.1.ts" header="src/app/heroes/heroes.component.ts" linenums="false">
</code-example>

### Element injectors

An injector does not actually belong to a component, but rather to the component instance's anchor element in the DOM. A different component instance on a different DOM element uses a different injector.

Components are a special type of directive, and the `providers` property of
`@Component()` is inherited from `@Directive()`. 
Directives can also have dependencies, and you can configure providers
in their `@Directive()` metadata. 
When you configure a provider for a component or directive using the `providers` property, that provider belongs to the injector for the anchor DOM element. Components and directives on the same element share an injector.

<!--- TBD with examples
* For an example of how this works, see [Element-level providers](guide/dependency-injection-in-action#directive-level-providers).
--->

* Learn more about [Element Injectors in Angular](https://blog.angularindepth.com/a-curios-case-of-the-host-decorator-and-element-injectors-in-angular-582562abcf0a).



## インジェクターバブリング

Consider this guide's variation on the Tour of Heroes application.
At the top is the `AppComponent` which has some subcomponents, such as the `HeroesListComponent`.
The `HeroesListComponent` holds and manages multiple instances of the `HeroTaxReturnComponent`.
The following diagram represents the state of this three-level component tree when there are three instances of `HeroTaxReturnComponent` open simultaneously.

<figure>
  <img src="generated/images/guide/dependency-injection/component-hierarchy.png" alt="injector tree">
</figure>

When a component requests a dependency, Angular tries to satisfy that dependency with a provider registered in that component's own injector.
If the component's injector lacks the provider, it passes the request up to its parent component's injector.
If that injector can't satisfy the request, it passes the request along to the next parent injector up the tree.
The requests keep bubbling up until Angular finds an injector that can handle the request or runs out of ancestor injectors.
If it runs out of ancestors, Angular throws an error. 

If you have registered a provider for the same DI token at different levels, the first one Angular encounters is the one it uses to provide the dependency. If, for example, a provider is registered locally in the component that needs a service, Angular doesn't look for another provider of the same service.  


<div class="alert is-helpful">

You can cap the bubbling by adding the `@Host()` parameter decorator on the dependant-service parameter
in a component's constructor. 
The hunt for providers stops at the injector for the host element of the component. 

* See an [example](guide/dependency-injection-in-action#qualify-dependency-lookup) of using `@Host` together with `@Optional`, another parameter decorator that lets you handle the null case if no provider is found.

* Learn more about the [`@Host` decorator and Element Injectors](https://blog.angularindepth.com/a-curios-case-of-the-host-decorator-and-element-injectors-in-angular-582562abcf0a).

</div>

If you only register providers with the root injector at the top level (typically the root `AppModule`), the tree of injectors appears to be flat.
All requests bubble up to the root injector, whether you configured it with the `bootstrapModule` method, or registered all providers with `root` in their own services.

{@a component-injectors}

## コンポーネントインジェクター

The ability to configure one or more providers at different levels opens up interesting and useful possibilities.
The guide sample offers some scenarios where you might want to do so.

### シナリオ: サービスの隔離

Architectural reasons may lead you to restrict access to a service to the application domain where it belongs. 
For example, the guide sample includes a `VillainsListComponent` that displays a list of villains.
It gets those villains from a `VillainsService`.

If you provide `VillainsService` in the root `AppModule` (where you registered the `HeroesService`),
that would make the `VillainsService` available everywhere in the application, including the _Hero_ workflows. If you later modified the `VillainsService`, you could break something in a hero component somewhere. Providing the service in the root `AppModule` creates that risk.

Instead, you can provide the `VillainsService` in the `providers` metadata of the `VillainsListComponent` like this:


<code-example path="hierarchical-dependency-injection/src/app/villains-list.component.ts" linenums="false" header="src/app/villains-list.component.ts (metadata)" region="metadata">

</code-example>

By providing `VillainsService` in the `VillainsListComponent` metadata and nowhere else,
the service becomes available only in the `VillainsListComponent` and its sub-component tree.
It's still a singleton, but it's a singleton that exist solely in the _villain_ domain.

Now you know that a hero component can't access it. You've reduced your exposure to error.

### シナリオ: 複数の編集セッション

Many applications allow users to work on several open tasks at the same time.
For example, in a tax preparation application, the preparer could be working on several tax returns,
switching from one to the other throughout the day.

This guide demonstrates that scenario with an example in the Tour of Heroes theme.
Imagine an outer `HeroListComponent` that displays a list of super heroes.

To open a hero's tax return, the preparer clicks on a hero name, which opens a component for editing that return.
Each selected hero tax return opens in its own component and multiple returns can be open at the same time.

Each tax return component has the following characteristics:

* Is its own tax return editing session.
* Can change a tax return without affecting a return in another component.
* Has the ability to save the changes to its tax return or cancel them.


<figure>
  <img src="generated/images/guide/dependency-injection/hid-heroes-anim.gif" alt="Heroes in action">
</figure>

Suppose that the `HeroTaxReturnComponent` has logic to manage and restore changes.
That would be a pretty easy task for a simple hero tax return.
In the real world, with a rich tax return data model, the change management would be tricky.
You could delegate that management to a helper service, as this example does.

Here is the `HeroTaxReturnService`.
It caches a single `HeroTaxReturn`, tracks changes to that return, and can save or restore it.
It also delegates to the application-wide singleton `HeroService`, which it gets by injection.


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.service.ts" header="src/app/hero-tax-return.service.ts">

</code-example>

Here is the `HeroTaxReturnComponent` that makes use of it.


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.component.ts" header="src/app/hero-tax-return.component.ts">

</code-example>


The _tax-return-to-edit_ arrives via the input property which is implemented with getters and setters.
The setter initializes the component's own instance of the `HeroTaxReturnService` with the incoming return.
The getter always returns what that service says is the current state of the hero.
The component also asks the service to save and restore this tax return.

This won't work if the service is an application-wide singleton.
Every component would share the same service instance, and each component would overwrite the tax return that belonged to another hero.

To prevent this, we configure the component-level injector of `HeroTaxReturnComponent` to provide the service, using the  `providers` property in the component metadata.


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.component.ts" linenums="false" header="src/app/hero-tax-return.component.ts (providers)" region="providers">

</code-example>

The `HeroTaxReturnComponent` has its own provider of the `HeroTaxReturnService`.
Recall that every component _instance_ has its own injector.
Providing the service at the component level ensures that _every_ instance of the component gets its own, private instance of the service, and no tax return gets overwritten.


<div class="alert is-helpful">


The rest of the scenario code relies on other Angular features and techniques that you can learn about elsewhere in the documentation.
You can review it and download it from the <live-example></live-example>.


</div>



### シナリオ: 特別なプロバイダー

Another reason to re-provide a service at another level is to substitute a _more specialized_ implementation of that service, deeper in the component tree.

Consider a Car component that depends on several services.
Suppose you configured the root injector (marked as A) with _generic_ providers for
`CarService`, `EngineService` and `TiresService`.

You create a car component (A) that displays a car constructed from these three generic services.

Then you create a child component (B) that defines its own, _specialized_ providers for `CarService` and `EngineService`
that have special capabilities suitable for whatever is going on in component (B).

Component (B) is the parent of another component (C) that defines its own, even _more specialized_ provider for `CarService`.


<figure>
  <img src="generated/images/guide/dependency-injection/car-components.png" alt="car components">
</figure>

Behind the scenes, each component sets up its own injector with zero, one, or more providers defined for that component itself.

When you resolve an instance of `Car` at the deepest component (C),
its injector produces an instance of `Car` resolved by injector (C) with an `Engine` resolved by injector (B) and
`Tires` resolved by the root injector (A).


<figure>
  <img src="generated/images/guide/dependency-injection/injector-tree.png" alt="car injector tree">
</figure>



<div class="alert is-helpful">


The code for this _cars_ scenario is in the `car.components.ts` and `car.services.ts` files of the sample
which you can review and download from the <live-example></live-example>.

</div>
