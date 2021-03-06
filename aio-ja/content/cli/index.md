# CLIの概要とコマンドリファレンス

Angular CLIは、Angularアプリケーションの初期化、開発、スキャフォールド、およびメンテナンスに使用するコマンドラインインターフェースツールです。このツールは、コマンドシェルで直接使用することも、 [Angularコンソール](https://angularconsole.com) などの対話型UIを通じて間接的に使用することもできます。

## Angular CLIのインストール

Angular CLIのメジャーバージョンは、サポートされているAngularのメジャーバージョンに従いますが、マイナーバージョンは個別にリリースできます。

`npm` パッケージマネージャーを使用してCLIをインストールします:
<code-example language="bash">
npm install -g @angular/cli
</code-example>

バージョン間の変更、および以前のリリースからのアップデートに関する詳細については、
GitHubのリリースタブを参照してください: https://github.com/angular/angular-cli/releases

## 基本的なワークフロー

コマンドラインから実行可能な `ng` を介してツールを起動します。
オンラインヘルプはコマンドラインで利用できます。
特定のコマンド（ [generate](cli/generate) など）またはオプションを簡単な説明付きでリストするには、次のように入力します。

<code-example language="bash">
ng help
ng generate --help
</code-example>

開発サーバーで新しい基本的なAngularプロジェクトを作成、構築、および提供するには、新しいワークスペースの親ディレクトリに移動し、次のコマンドを使用します:

<code-example language="bash">
ng new my-first-project
cd my-first-project
ng serve
</code-example>

ブラウザで http://localhost:4200/ を開き、新しいアプリが実行されるのを確認します。
[ng serve](cli/serve) コマンドを使用してアプリを構築し、それをローカルに配信すると、ソースファイルのいずれかを変更すると、サーバーによって自動的にアプリが再構築され、ページがリロードされます。

<div class="alert is-helpful">

   When you run `ng new my-first-project` a new folder, named `my-first-project`, will be created in the current working directory. Since you want to be able to create files inside that folder, make sure you have sufficient rights in the current working directory before running the command.

   If the current working directory is not the right place for your project, you can change to a more appropriate directory by running `cd <path-to-other-directory>` first.

</div>

## ワークスペースとプロジェクトファイル

[ng new](cli/new) コマンドは *Angular workspace* フォルダを作成し、新たなアプリケーションのスケルトンを生成します。
ワークスペースには複数のアプリとライブラリを含めることができます。
[ng new](cli/new) コマンドによって作成された最初のアプリは、ワークスペースの最上位にあります。
ワークスペースで追加のアプリまたはライブラリを生成すると、それらは `projects/` サブフォルダに入ります。

新しく生成されたアプリには、ルートコンポーネントとテンプレートを含むルートモジュールのソースファイルが含まれています。
各アプリにはロジック、データ、およびアセットを含む `src` フォルダがあります。

生成されたファイルを直接編集することも、CLIコマンドを使用して追加して変更することもできます。
[ng generate](cli/generate) コマンドを使用して、追加のコンポーネントやサービス用の新しいファイルを追加したり、新しいパイプやディレクティブなどのコードを追加したりします。
アプリやライブラリを作成または操作する [add](cli/add) や [generate](cli/generate) などのコマンドは、ワークスペースまたはプロジェクトフォルダー内から実行する必要があります。

* [ワークスペースとプロジェクトのファイル構造](guide/file-structure) についての詳細を参照してください。

### ワークスペースとプロジェクト構成

ひとつのワークスペース設定ファイル `angular.json` が、ワークスペースの最上位に作成されます。
ここで、CLIコマンドオプションにプロジェクトごとのデフォルトを設定し、CLIがさまざまなターゲット用にプロジェクトをビルドするときに使用する構成を指定できます。

[ng config](cli/config) コマンドを使用すると、コマンドラインから設定値を設定および取得できます。または、 `angular.json` ファイルを直接編集できます。
設定ファイルのオプション名には [camelCase](guide/glossary#case-types) を使用する必要がありますが、コマンドに指定するオプション名にはcamelCaseまたはダッシュケースを使用できます。

* [ワークスペースの設定](guide/workspace-config) ワークスペース設定についての詳細を参照してください。
* `angular.json`の [完全なスキーマ](https://github.com/angular/angular-cli/wiki/angular-workspace) を参照してください。

## CLIコマンド言語構文

コマンド構文は次のとおりです:

`ng` *commandNameOrAlias* *requiredArg* [*optionalArg*] `[options]`

* ほとんどのコマンドと一部のオプションにはエイリアスがあります。別名は、各コマンドの構文ステートメントに示されています。

* オプション名の前には二重ダッシュ（--）が付きます。
    オプションエイリアスの先頭には単一ダッシュ（ - ）が付きます。
    引数は前に付けられません。
    たとえば: 
    <code-example language="bash">
        ng build my-app -c production
    </code-example>

* 通常、生成された成果物の名前は、コマンドの引数として指定することも、--nameオプションで指定することもできます。

* 引数とオプションの名前は
[camelCaseかdash-case](guide/glossary#case-types)で指定できます。
`--myOptionName` は `--my-option-name` と同じです。

### ブール値および列挙型オプション

ブール値オプションには2つの形式があります: `--thisOption` はフラグを設定し、 `--noThisOption` はフラグをクリアします。
どちらのオプションも指定されていない場合、フラグはリファレンスドキュメントに記載されているデフォルトのままになります。

許容値は、列挙型オプションの説明ごとに示されており、デフォルト値は **太字** で示されています。

### 相対パス

ファイルを指定するオプションは、絶対パスとして、または現在の作業ディレクトリ（通常はワークスペースまたはプロジェクトルートのどちらか）に対する相対パスとして指定できます。

### Schematics

[ng generate](cli/generate) と [ng add](cli/add) コマンドは、アーティファクトやライブラリが生成されるか、現在のプロジェクトに追加する引数として取ります
一般的なオプションに加えて、各アーティファクトまたはライブラリは、 *schematics* で独自のオプションを定義します。
Schematicオプションは、即時コマンドオプションと同じ形式でコマンドに提供されます。


### Bazelを使ったビルド

オプションで [Bazel](https://docs.bazel.build) をビルドツールとして使用するようにAngular CLIを設定できます。詳しくは [Building with Bazel](guide/bazel) を参照してください。
