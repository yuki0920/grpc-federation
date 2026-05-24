# SWAPI デモ 操作メモ

`demo/` 配下のデモを起動・停止・grpcurl で叩くためのコマンド集。

## 共通の前提（毎回必要）

```sh
cd demo
export DOCKER_HOST=unix://$HOME/.docker/run/docker.sock
```

> 毎回打つのが面倒なら `direnv` を使うか、シェルの rc に書いてしまうのが楽。

## 起動

| 用途 | コマンド |
|---|---|
| 起動 (フォアグラウンド、ログ流す) | `make up` |
| 起動 (バックグラウンド) | `docker compose up -d` |
| 状態確認 | `docker compose ps` |
| ログ確認 | `docker compose logs -f swapi` （サービス名を変えれば各 microservice のログ） |

> 初回 or コードを変更した後は `make generate && make build` を先に。

## 停止

| 用途 | コマンド |
|---|---|
| フォアグラウンド起動を止める | `Ctrl+C` |
| コンテナ停止＋削除（推奨） | `docker compose down` |
| 一時停止だけ（コンテナは残す） | `docker compose stop` |
| 再開 | `docker compose start` |

## grpcurl での動作確認

`grpcurl` は以下のいずれかを使う想定。コマンド例では `grpcurl` と表記（必要に応じて `./bin/grpcurl` に読み替え）。

- `make tools` で `demo/bin/grpcurl` を入れて `./bin/grpcurl` として実行（`demo/` 配下で）
- 既に `$GOPATH/bin` 等に `grpcurl` が入っているならそれを利用

### サービス一覧 / メソッド一覧（reflection）

```sh
grpcurl -plaintext localhost:3000 list
grpcurl -plaintext localhost:3000 list swapi.SWAPI
grpcurl -plaintext localhost:3000 describe swapi.SWAPI.GetPerson
```

### 単体取得

```sh
grpcurl -plaintext -d '{"id": 1}' localhost:3000 swapi.SWAPI/GetPerson
grpcurl -plaintext -d '{"id": 1}' localhost:3000 swapi.SWAPI/GetFilm
grpcurl -plaintext -d '{"id": 1}' localhost:3000 swapi.SWAPI/GetPlanet
grpcurl -plaintext -d '{"id": 1}' localhost:3000 swapi.SWAPI/GetStarship
grpcurl -plaintext -d '{"id": 1}' localhost:3000 swapi.SWAPI/GetSpecies
grpcurl -plaintext -d '{"id": 1}' localhost:3000 swapi.SWAPI/GetVehicle
```

### 一覧取得（List 系）

```sh
grpcurl -plaintext -d '{}' localhost:3000 swapi.SWAPI/ListPeople
grpcurl -plaintext -d '{}' localhost:3000 swapi.SWAPI/ListFilms
grpcurl -plaintext -d '{}' localhost:3000 swapi.SWAPI/ListPlanets
grpcurl -plaintext -d '{}' localhost:3000 swapi.SWAPI/ListStarships
grpcurl -plaintext -d '{}' localhost:3000 swapi.SWAPI/ListSpecies
grpcurl -plaintext -d '{}' localhost:3000 swapi.SWAPI/ListVehicles
```

### トレース確認

ブラウザで http://localhost:4000 を開く → Service に `swapi` を選択して Find Traces。

## トラブル時のチェック

```sh
docker compose ps              # 全コンテナ Up になっているか
docker compose logs swapi      # swapi ログ
docker compose logs person     # 各 microservice のログ
```

## 補足

- 今回入れた `InsecureSkipVerify` の修正（`util/util.go` の `NewSwapiClient`）は swapi.dev の証明書が再発行されたら不要になる類のもの。本家のフィックスではないので、コミット前に意識すること。
- Go ツールチェイン: `GOTOOLCHAIN=go1.24.13` が必要なのは `make tools` のときだけ。一度入れた `bin/` のバイナリには関係なし。
