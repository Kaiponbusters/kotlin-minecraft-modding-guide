# Kotlin + Fabric Minecraft MOD開発FAQ

## 環境準備の問題

### Q: Java 8を指定しているのに「Java 16以上が必要」というエラーが出る
**A:** Minecraft 1.17以降はJava 16以上が必要です。以下の設定を確認してください：
1. IntelliJ IDEAの`File > Project Structure`で、Project SDKを16以上に設定
2. `File > Settings > Build, Execution, Deployment > Build Tools > Gradle`で、Gradle JVMを16以上に設定
3. 環境変数`JAVA_HOME`がJava 16以上のインストールパスを指していることを確認
4. `build.gradle`または`build.gradle.kts`ファイルで、Java Version/Toolchainを確認

### Q: Gradleタスクの実行中にエラーが発生する
**A:** 以下を試してみてください：
1. `gradlew cleanloom`を実行してキャッシュをクリア
2. `gradlew --stop`を実行してGradleデーモンを停止
3. `build.gradle`や`gradle.properties`で依存関係のバージョンが適切か確認
4. Gradle自体のバージョンが古い場合は`gradle/wrapper/gradle-wrapper.properties`を更新

### Q: IntelliJ IDEAでプロジェクトをインポートできない
**A:** 以下を確認してください：
1. Gradleプラグインが有効になっているか
2. `.idea`フォルダを削除して新たにインポートしてみる
3. `build.gradle`または`build.gradle.kts`ファイルを開いてからインポートする
4. Gradleタブから「Refresh」または「Sync」ボタンを押す

## Kotlin固有の問題

### Q: ModInitializerが見つからないというエラーが出る
**A:** `fabric.mod.json`ファイルのエントリポイント設定が正しいか確認してください。Kotlinクラスの場合、以下のように設定する必要があります：
```json
"entrypoints": {
  "main": [
    {
      "adapter": "kotlin",
      "value": "com.example.examplemod.ExampleMod"
    }
  ]
}
```

### Q: Kotlinのトップレベル関数をエントリポイントとして使用したい
**A:** ファイル名が`MyMod.kt`の場合、以下のように設定します：
```json
"entrypoints": {
  "main": [
    {
      "adapter": "kotlin",
      "value": "com.example.mymod.MyModKt::init"
    }
  ]
}
```
ここで`init`はトップレベル関数の名前です。

### Q: 「Fabric Language Kotlin」がないというエラーが出る
**A:** 以下を確認してください：
1. `build.gradle`または`build.gradle.kts`に依存関係が記述されているか
   ```gradle
   dependencies {
     modImplementation("net.fabricmc:fabric-language-kotlin:1.9.0+kotlin.1.8.0")
   }
   ```
2. `fabric.mod.json`の依存関係に記述されているか
   ```json
   "depends": {
     "fabric-language-kotlin": ">=1.9.0+kotlin.1.8.0"
   }
   ```

### Q: MixinをKotlinで書きたい
**A:** MixinはJavaで書く必要があります。KotlinでMixinを直接書くことはできませんが、以下の方法で対処できます：
1. Mixinのクラス自体はJavaで書く
2. Kotlin側に`@JvmStatic`でアノテートされたメソッドを持つ`object`や`companion object`を作る
3. Javaの Mixin からそのメソッドを呼び出す

## MOD開発における一般的な問題

### Q: アイテムを追加したがゲーム内に表示されない
**A:** 以下をチェックしてください：
1. アイテムが正しく登録されているか（`Registry.register`メソッドの呼び出し）
2. テクスチャパスが正しいか（`assets/[modid]/textures/item/[item_id].png`）
3. モデルJSONが正しいか（`assets/[modid]/models/item/[item_id].json`）
4. 言語ファイルに翻訳が追加されているか（`assets/[modid]/lang/en_us.json`）

### Q: テクスチャが紫黒のチェック柄になる
**A:** テクスチャが見つからない状態です。以下をチェックしてください：
1. テクスチャファイルの場所が正しいか
2. モデルJSONでのテクスチャ参照パスが正しいか
3. テクスチャ名が小文字かつスペースを含まないか
4. MOD IDが正確に一致しているか

### Q: クラフトレシピが機能しない
**A:** 以下をチェックしてください：
1. レシピJSONの構文が正しいか
2. アイテムIDが正確に一致しているか（`minecraft:`または`[modid]:`の接頭辞を確認）
3. ファイルパスが正しいか（`data/[modid]/recipes/[recipe_id].json`）
4. レシピタイプが正しいか（`crafting_shaped`、`crafting_shapeless`など）

### Q: ブロックを壊してもドロップしない
**A:** 以下をチェックしてください：
1. ルートテーブルが正しく設定されているか（`data/[modid]/loot_tables/blocks/[block_id].json`）
2. ブロックの`dropsNothing()`メソッドが呼ばれていないか
3. ツールの要件が正しく設定されているか（`requiresTool()`など）

### Q: コード変更が反映されない
**A:** 以下を試してみてください：
1. プロジェクトを再ビルド（`gradlew build`）
2. Minecraftクライアントを再起動
3. IntelliJ IDEAの場合、`Build > Rebuild Project`を実行
4. 問題が解決しない場合は、`gradlew clean build`を実行してビルドをクリーンアップ

## バージョンアップデートの問題

### Q: Minecraftの新バージョンに対応するには？
**A:** 以下の手順で対応します：
1. `gradle.properties`のMinecraftバージョン、Yarn Mappings、Loader Versionを更新
2. Fabric APIやFabric Language Kotlinなど依存関係のバージョンも更新
3. コード内のAPIの変更点を確認し、必要に応じて修正
4. メジャーアップデートの場合、クラス名やメソッド名が変更されている可能性があるため注意

### Q: APIのバージョンが多すぎて混乱する
**A:** 以下のリソースを活用して最新のバージョン情報を確認してください：
1. [Fabric Versions](https://fabricmc.net/versions.html) - 公式の最新バージョン情報
2. [ModUpdater](https://github.com/comp500/ModUpdater-Fabric) - 依存関係の更新を支援するツール
3. 人気のあるMODのリポジトリを参考にする

## パフォーマンスと互換性の問題

### Q: MODがサーバー側でクラッシュする
**A:** 以下を確認してください：
1. クライアント専用のコードがサーバー側で実行されていないか
2. `fabric.mod.json`の`environment`設定が適切か（`*`、`client`、`server`）
3. クライアント/サーバー側の処理を適切に分離しているか
4. サーバーがMODの要件を満たしているか（Java バージョンなど）

### Q: 他のMODと競合する
**A:** 以下の対策を検討してください：
1. MOD IDが他のMODと被っていないことを確認
2. レジストリ名（アイテムID、ブロックIDなど）が重複していないか確認
3. 共通のインターフェースやAPIを利用して互換性を向上させる
4. 他のMODを改変するコードはMixinを適切に使用する

### Q: MODのパフォーマンスが悪い
**A:** 以下の最適化を検討してください：
1. ティック毎の処理を最小限に抑える
2. 不要なオブジェクト生成を避ける
3. 大量のブロック更新を一度に行わない
4. レンダリングコードを最適化する
5. プロファイリングツールを使ってボトルネックを特定する

## その他の問題

### Q: MODに国際化対応を追加するには？
**A:** 以下の手順で対応します：
1. 各言語の翻訳ファイルを`assets/[modid]/lang/`ディレクトリに追加
   - 英語: `en_us.json`
   - 日本語: `ja_jp.json`
   - など
2. 文字列を直接コードに書くのではなく、翻訳キーを使用
   ```kotlin
   Text.translatable("item.[modid].[item_id].name")
   ```

### Q: MODの依存関係を管理する良い方法は？
**A:** 以下の方法を検討してください：
1. 必須の依存関係は`fabric.mod.json`の`depends`に記載
2. オプションの依存関係は`suggests`に記載
3. 特定のバージョン範囲を指定（例：`>=1.0.0`、`~1.0.0`）
4. 複雑な依存関係はModMenuなどのMOD管理MODと連携

### Q: MODのデバッグ方法は？
**A:** 以下の方法を活用してください：
1. ログ出力を活用（SLF4Jロガーを使用）
2. IntelliJ IDEAのデバッガでブレークポイントを設定
3. `runClient`タスクでデバッグモードで実行
4. クラッシュレポートを詳細に分析
5. 開発環境でのみ有効になるデバッグコードを追加

### Q: MODのライセンスはどうすればいい？
**A:** 以下を検討してください：
1. オープンソースライセンス（MIT、Apache、GPLなど）の理解
2. アセット（テクスチャ、サウンド等）のライセンスも明確にする
3. 使用している他のMODやライブラリのライセンスとの互換性を確認
4. `LICENSE`ファイルをプロジェクト（およびJAR）に含める
5. 必要に応じて`CREDITS.md`ファイルでクレジットを記載

### Q: 効果的な学習方法は？
**A:** 以下の方法を試してみてください：
1. 既存のオープンソースMODのコードを読む
2. 小さな機能から始めて徐々に複雑にする
3. Fabric Discord サーバーなどコミュニティに参加する
4. Fabricのバグトラッカーやwikiを参照する
5. Minecraftのソースコード（mappingsされたもの）を読む習慣をつける

ご質問があれば、FabricのコミュニティフォーラムやDiscordサーバーで質問してください。多くの経験豊富な開発者が支援してくれるでしょう。 