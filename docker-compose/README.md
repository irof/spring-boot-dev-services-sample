# Spring BootのDocker Composeサポートのサンプル実装

- [Spring Boot / リファレンス / コア機能 / 開発時のサービス / Docker Compose サポート](https://spring.pleiades.io/spring-boot/reference/features/dev-services.html#features.dev-services.docker-compose)
- [Spring Boot / リファレンス / コア機能 / 開発時のサービス / Docker Compose サポート / テストでの Docker Compose の使用](https://spring.pleiades.io/spring-boot/reference/features/dev-services.html#features.dev-services.docker-compose.tests)

## ざっくり導入手順

- `compose.yml` を用意
- 依存に `testAndDevelopmentOnly("org.springframework.boot:spring-boot-docker-compose")` を追加
- テストで使う場合は `spring.docker.compose.skip.in-tests=false` を指定

## ローカル実行時のDocker Compose使用

```shell
../gradlew bootRun
```

[build.gradle.kts](build.gradle.kts) で依存に `org.springframework.boot:spring-boot-docker-compose` を追加するだけで、
working directoryの `compose.yml` を使ってくれます。
リリースモジュールに入らないように `developmentOnly` にするのが良いでしょう。

サンプルでは後述の理由でクラスパス上の `dev-compose.yml` を使うように設定しています。

## テストでのDocker Compose使用

ローカル実行とテストともにDocker Composeサポートを使用する場合は `testAndDevelopmentOnly` とした上で、
`spring.docker.compose.skip.in-tests=false` を指定する必要があります。
サンプルのようにプロファイルにしてもよいですし、合成アノテーションを使ってもいいと思います。

サンプルでは後述の理由でクラスパス上の `test-compose.yml` を使うように設定しています。

テスト時のみで使用する場合は `testAndDevelopmentOnly` ではなく `testRuntimeOnly` でいいと思います。

## メモ

`compose.yml` の検出はworking directoryを起点に行いますが、working directoryは実行の仕方によって変わります。
たとえば`MyApplication`をIDEで実行した場合、未設定だとルートプロジェクトがworking directoryになったりしますが、
テストを実行した場合はプロジェクトルートがworking directoryになったりします。

このサンプルではマルチプロジェクト構成で動作を安定させるために `spring.docker.compose.file=classpath:dev-compose.yml`
のようにクラスパスで指定しています。（ [application.yml](./src/main/resources/application.yml) 参照）

テストでは `classpath:test-compose.yml` を使用しているのは別設定も使用できることを示していますが、Docker Composeの設定が複数になるのも微妙ですし、
同じにできるなら同じものを使う方が好ましいです。
クラスパスのものを使う場合、 `main` に置くとリリースモジュールに含まれることになるのは少し気になるところです。
運用環境で使われはしないので害はありませんが、気になるなら除外しましょう。

Docker Composeを開発時のアプリケーション起動やテスト時以外でも起動するのであればデフォルトのままworking directoryを使用するとして、
うまく動作しない時はworking directoryを設定するか、 `spring.docker.compose.file` を絶対パスで指定するのが良いでしょう。
