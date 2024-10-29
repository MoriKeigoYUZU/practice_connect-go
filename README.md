# 概要
[connect-go 公式ドキュメントの写し](https://connectrpc.com/docs/go/getting-started)

# ハンズオン
## Goプロジェクトの作成
`gRPC`サービスを構築するため、まず`Go`プロジェクトを初期化します。
`Go`では、プロジェクトの依存関係やモジュール管理に`go mod`を使用します。

```zsh
(go mod init [プロジェクト名])
go mod init example
```

## ディレクトリとprotoファイルの作成
`gRPC`サービスで使用する `Protocol Buffers（.proto）`ファイル を作成します。
このファイルには、`API`のリクエスト・レスポンスの形式や、`RPC`メソッドの定義を記述します。

```zsh
mkdir -p greet/v1
touch greet/v1/greet.proto
```

## greet.protoの定義
次に、`greet.proto`に`gRPC`サービスの定義を書き込みます。
`greet.proto`ファイルでは、リクエスト・レスポンスのメッセージ形式と、それを使うサービスメソッドを定義します。

greet.protoの内容
```protobuf
syntax = "proto3";

package greet.v1;

option go_package = "example/gen/greet/v1;greetv1";

message GreetRequest {
  string name = 1;
}

message GreetResponse {
  string greeting = 1;
}

service GreetService {
  rpc Greet(GreetRequest) returns (GreetResponse) {}
}
```

### 解説
1. `syntax = "proto3";`
   - `Proto3`は`Protocol Buffers`の最新バージョンで、シンプルかつ軽量なデータ形式で定義を行います。

2. `package greet.v1;`
   - パッケージは、`API`を論理的に分割し、命名の競合を避けるために使います。ここでは`greet`のバージョン1（v1）を表しています。


3. `option go_package`
   - 生成される`Go`コードの出力先を指定します。`example/gen/greet/v1;greetv1`の形式で、`greetv1`という名前でGoパッケージとして利用できるようになります。


4. `message GreetRequest`
   - リクエストの形式を表すメッセージです。ここでは、`name`という文字列を含む構造体を定義しています。


5. `message GreetResponse`
   - レスポンスの形式を表すメッセージです。`greeting`という文字列を返します。


6. `service GreetService`
   - `gRPC`のサービスを定義するブロックです。
   - `rpc Greet`はリクエストとして`GreetRequest`を受け取り、`GreetResponse`を返すRPCメソッドです。

## Bufモジュールの初期化
次に、Bufを使ってモジュールを初期化します。
これにより、`buf.yaml`という設定ファイルが生成されます。

```zsh
buf mod init
```

### buf.yamlの解説
`buf mod init`コマンドを実行すると、次のような`buf.yaml`が生成されます。

```yaml
# For details on buf.yaml configuration, visit https://buf.build/docs/configuration/v2/buf-yaml
version: v2
lint:
  use:
    - STANDARD
breaking:
  use:
    - FILE
```

### 解説
1. `version: v2`
   - Bufの設定ファイルのバージョンを示します。
   - Bufのルールセットや構文チェックの対象が、このバージョンに基づいて動作します。
2. lintセクション `use: - STANDARD`
   - Bufが推奨する 「標準的なLintルール」 を適用します。
   - これにより、Protoファイルの構文エラーやフォーマットのミスを検出し、プロジェクトの品質を保ちます。
3. breakingセクション `use: - FILE`
   - ファイル単位での**Breaking Change（互換性を壊す変更）** を検出します。
   - 例えば、ProtoファイルからメッセージやRPCメソッドを削除するなどの変更を検出し、意図せずAPIが壊れるのを防ぎます。

### Bufを使ったProtoファイルのチェック
`buf.yaml`ファイルが設定された状態で、`lint`チェックやビルドを行えます。


```zsh:Lintチェックの実行
buf lint
```
このコマンドを実行すると、`greet.proto`の構文やスタイルに問題がないかBufがチェックします。

```zsh:ビルドの実行
buf build
```
このコマンドで、Bufが`Proto`ファイルをビルドし、エラーがないか確認します。

## buf.gen.yamlの作成
まず、コード生成のための`buf.gen.yaml`を作成します。

```zsh
touch buf.gen.yaml
```

`buf.gen.yaml` の内容
(環境に合わせて調整してください)
```yaml
version: v2
plugins:
- local: protoc-gen-go
  out: gen
  opt: paths=source_relative
- local: protoc-gen-connect-go
  out: gen
  opt: paths=source_relative
```

### 解説
1. `version: v2`
   - Bufの設定が最新のv2形式であることを示します。

2. `plugins`
   - local：ローカルにインストールされたプラグインを使用します。
      - `protoc-gen-go`：Goのコードを生成するためのプラグイン。
      - `protoc-gen-connect-go`：Connect-goのためのハンドラコードを生成します。

3. `out: gen`
   - 生成されたコードの出力先を`gen/`ディレクトリに指定します。

4. `opt: paths=source_relative`
   - ファイルが相対パスで生成され、ソースコード内のパッケージ構造に合致するようにします。

## コード生成の実行
`greet.proto`からGoコードを生成します。

```zsh
buf generate
```

gRPCサービスを実装するためのコード生成が完了しました。

### 生成されるコードの内容
greet.proto に基づいて以下の2種類のコードが生成されます。

1. `protoc-gen-go`によるコード生成

ファイル: `/gen/greet/v1/greet.pb.go`

役割：`gRPC`のメッセージ型やサービスの定義を`Go`用の構造体として生成します。
`GreetRequest`や`GreetResponse`がGoの型として使えるようになります。
gRPCサーバー側で使われるサービスインターフェースも定義されます。

2. `protoc-gen-connect-go`によるコード生成

ファイル: `/gen/greet/v1/greetv1connect/greet.connect.go`

役割：`Connect-go`用のハンドラコードを生成します。
`GreetService`のサービスハンドラが自動生成され、`Go`サーバーで使う`RPC`エンドポイントを簡単にセットアップできます。
`Connect-go`は、`gRPC`だけでなく、`gRPC-Web`や`HTTP/JSON`にも対応した柔軟な通信をサポートします。

## Goサーバーの作成
生成された `gRPC`メッセージ・ハンドラコードを使って、`Go`サーバーを実装します。
このサーバーは、`Connect-go`を使って`gRPC`リクエストを受け取り、`HTTP/2`で通信を行います。

### ディレクトリ構成
次のように、サーバーのエントリーポイントとなる`Go`ファイルを作成します。

```zsh
mkdir -p cmd/server
touch cmd/server/main.go
```

### Goサーバー実装：cmd/server/main.go

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"

	"github.com/bufbuild/connect-go" // 注意:公式では"connectrpc.com/connect"となっていますが、動作しないです。
	"golang.org/x/net/http2"
	"golang.org/x/net/http2/h2c"

	greetv1 "example/gen/greet/v1"        // generated by protoc-gen-go
	"example/gen/greet/v1/greetv1connect" // generated by protoc-gen-connect-go
)

type GreetServer struct{}

func (s *GreetServer) Greet(
	ctx context.Context,
	req *connect.Request[greetv1.GreetRequest],
) (*connect.Response[greetv1.GreetResponse], error) {
	log.Println("Request headers: ", req.Header())
	res := connect.NewResponse(&greetv1.GreetResponse{
		Greeting: fmt.Sprintf("Hello, %s!", req.Msg.Name),
	})
	res.Header().Set("Greet-Version", "v1")
	return res, nil
}

func main() {
	greeter := &GreetServer{}
	mux := http.NewServeMux()
	path, handler := greetv1connect.NewGreetServiceHandler(greeter)
	mux.Handle(path, handler)
	http.ListenAndServe(
		"localhost:8080",
		// Use h2c so we can serve HTTP/2 without TLS.
		h2c.NewHandler(mux, &http2.Server{}),
	)
}
```

### 解説
1. `GreetServer`構造体
   - `GreetServer`は、`GreetService`のメソッドを実装する構造体です。
   - `Greet`メソッドで、リクエストからユーザー名を取得し、`Hello, [name]!`という挨拶をレスポンスとして返します。

2. `connect-go`の使用
   - `github.com/bufbuild/connect-go`をインポートして使います。公式では`connectrpc.com/connect`というパスが推奨されていますが、互換性の問題があるため`bufbuild`のパスを使います。

3. `h2c`を使った`HTTP/2`の起動
- `h2c.NewHandler`を使い、`TLS`なしで`HTTP/2`サーバーを起動しています。
- `gRPC`は`HTTP/2`をベースとしているため、この設定が必要です。

4. HTTPハンドラへのサービス登録
   - `greetv1connect.NewGreetServiceHandler`を使って、`GreetService`をHTTPハンドラとして登録します。
   - このハンドラは、`gRPC-Web`や`HTTP/JSON`にも対応できます。

## go mod tidy
最後に
```zsh
go mod tidy
```
をしてください。

## サーバーの実行とテスト
`go run`コマンドでサーバーを起動し、`curl`と`grpcurl`を使ってサービスが正しく応答するか確認します。

### サーバーの起動
```zsh
go run ./cmd/server/main.go
```

### curlでのテスト
`curl`を使って、サービスにリクエストを送信し、JSONレスポンスが正常に返ってくるか確認します。

```zsh
curl \
    --header "Content-Type: application/json" \
    --data '{"name": "Jane"}' \
    http://localhost:8080/greet.v1.GreetService/Greet
```

実行結果
```zsh
{
  "greeting": "Hello, Jane!"
}
```

### grpcurlでのテスト
`grpcurl`を使ってgRPC形式でリクエストを送信します。
```zsh
grpcurl \
    -protoset <(buf build -o -) -plaintext \
    -d '{"name": "Jane"}' \
    localhost:8080 greet.v1.GreetService/Greet
```

実行結果
```zsh
{
  "greeting": "Hello, Jane!"
}
```