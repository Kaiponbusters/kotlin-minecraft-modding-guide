# Fabricを使ったKotlinによるMinecraft MOD開発サンプル

以下では、Kotlin言語を使用してFabricでMinecraft MODを開発する基本的なサンプルをステップバイステップで解説します。

## 1. 環境準備

### JDKのインストール
- AdoptOpenJDK 17以上を[公式サイト](https://adoptium.net/)からダウンロードしてインストール
- 環境変数`JAVA_HOME`を設定して、Javaコマンドがパスに含まれていることを確認

### IntelliJ IDEAのインストール
- [公式サイト](https://www.jetbrains.com/idea/download/)からCommunity Editionをダウンロードしてインストール
- Minecraft開発サポートのためのプラグイン「Minecraft Development」をインストール

### Fabricテンプレートの取得
- [Fabric Template Generator](https://fabricmc.net/develop/template/)にアクセス
- 以下の設定でテンプレートを生成:
  - Mod Name: ExampleMod
  - Package Name: com.example.examplemod
  - Minecraft Version: 最新バージョン
  - Kotlin Programming Language: ✓
  - Data Generation: ✓ (あれば)
  - Split client and common sources: ✓ (あれば)

## 2. プロジェクトの設定

### プロジェクトの読み込み
1. IntelliJ IDEAを起動
2. ダウンロードしたテンプレートを解凍
3. `build.gradle.kts`または`build.gradle`ファイルを開いてプロジェクトをインポート
4. Gradleの設定を以下のように変更:
   - Settings > Build, Execution, Deployment > Build Tools > Gradle
   - 'Build and run using' と 'Run tests using' を 'IntelliJ IDEA' に設定

### Minecraft資材の生成
1. IDEAのGradleタブを開く
2. Tasks > fabric > genSources をダブルクリックして実行
3. Minecraft資材の生成を待つ (時間がかかることがあります)

## 3. 基本的なMODの設定

### MOD情報の更新
ファイル `src/main/resources/fabric.mod.json` を開き、以下の情報を更新:

```json
{
  "schemaVersion": 1,
  "id": "examplemod",
  "version": "${version}",
  "name": "Example Mod",
  "description": "This is an example mod",
  "authors": [
    "Your Name"
  ],
  "contact": {
    "homepage": "https://example.com/",
    "sources": "https://github.com/yourusername/examplemod"
  },
  "license": "MIT",
  "icon": "assets/examplemod/icon.png",
  "environment": "*",
  "entrypoints": {
    "main": [
      {
        "adapter": "kotlin",
        "value": "com.example.examplemod.ExampleMod"
      }
    ]
  },
  "depends": {
    "fabricloader": ">=0.14.0",
    "fabric-api": "*",
    "minecraft": "~1.20",
    "java": ">=17",
    "fabric-language-kotlin": ">=1.9.0+kotlin.1.8.0"
  }
}
```

### MODのメインクラスの実装
ファイル `src/main/kotlin/com/example/examplemod/ExampleMod.kt` を開き、以下のように実装:

```kotlin
package com.example.examplemod

import net.fabricmc.api.ModInitializer
import org.slf4j.LoggerFactory

class ExampleMod : ModInitializer {
    private val logger = LoggerFactory.getLogger("examplemod")

    override fun onInitialize() {
        logger.info("Example Mod has been initialized!")
        
        // アイテムの登録
        ModItems.register()
        
        // ブロックの登録
        ModBlocks.register()
    }
    
    companion object {
        const val MOD_ID = "examplemod"
    }
}
```

## 4. カスタムアイテムの追加

### アイテムクラスの作成
新しいファイル `src/main/kotlin/com/example/examplemod/ModItems.kt` を作成:

```kotlin
package com.example.examplemod

import net.fabricmc.fabric.api.item.v1.FabricItemSettings
import net.minecraft.item.Item
import net.minecraft.item.ItemGroup
import net.minecraft.registry.Registries
import net.minecraft.registry.Registry
import net.minecraft.util.Identifier

object ModItems {
    // 通常のアイテム
    val EXAMPLE_ITEM = Item(FabricItemSettings())
    
    // カスタム機能を持つアイテム
    val SPECIAL_ITEM = SpecialItem(FabricItemSettings())
    
    fun register() {
        Registry.register(
            Registries.ITEM,
            Identifier(ExampleMod.MOD_ID, "example_item"),
            EXAMPLE_ITEM
        )
        
        Registry.register(
            Registries.ITEM,
            Identifier(ExampleMod.MOD_ID, "special_item"),
            SPECIAL_ITEM
        )
    }
}
```

### カスタム機能を持つアイテムの実装
新しいファイル `src/main/kotlin/com/example/examplemod/SpecialItem.kt` を作成:

```kotlin
package com.example.examplemod

import net.minecraft.client.item.TooltipContext
import net.minecraft.entity.player.PlayerEntity
import net.minecraft.item.Item
import net.minecraft.item.ItemStack
import net.minecraft.sound.SoundEvents
import net.minecraft.text.Text
import net.minecraft.util.ActionResult
import net.minecraft.util.Hand
import net.minecraft.util.TypedActionResult
import net.minecraft.world.World

class SpecialItem(settings: Settings) : Item(settings) {
    
    // アイテムを使用したときの処理
    override fun use(world: World, user: PlayerEntity, hand: Hand): TypedActionResult<ItemStack> {
        // 効果音を鳴らす
        user.playSound(SoundEvents.ENTITY_EXPERIENCE_ORB_PICKUP, 1.0f, 1.0f)
        
        // クールダウンを設定 (20 ticks = 1秒)
        user.itemCooldownManager.set(this, 20)
        
        return TypedActionResult.success(user.getStackInHand(hand))
    }
    
    // ツールチップ表示
    override fun appendTooltip(
        stack: ItemStack,
        world: World?,
        tooltip: MutableList<Text>,
        context: TooltipContext
    ) {
        tooltip.add(Text.translatable("item.examplemod.special_item.tooltip"))
        super.appendTooltip(stack, world, tooltip, context)
    }
}
```

## 5. テクスチャとモデルの追加

### アイテムテクスチャの追加
1. `src/main/resources/assets/examplemod/textures/item/` ディレクトリを作成
2. 以下の2つのPNGファイルを作成:
   - `example_item.png`
   - `special_item.png`

### アイテムモデルの追加
1. `src/main/resources/assets/examplemod/models/item/` ディレクトリを作成
2. `example_item.json` ファイルを作成:

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "examplemod:item/example_item"
  }
}
```

3. `special_item.json` ファイルを作成:

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "examplemod:item/special_item"
  }
}
```

### 言語ファイルの追加
`src/main/resources/assets/examplemod/lang/en_us.json` ファイルを作成:

```json
{
  "item.examplemod.example_item": "Example Item",
  "item.examplemod.special_item": "Special Item",
  "item.examplemod.special_item.tooltip": "Right-click to make a sound!"
}
```

## 6. 基本的なブロックの追加

### ブロッククラスの作成
新しいファイル `src/main/kotlin/com/example/examplemod/ModBlocks.kt` を作成:

```kotlin
package com.example.examplemod

import net.fabricmc.fabric.api.item.v1.FabricItemSettings
import net.fabricmc.fabric.api.`object`.builder.v1.block.FabricBlockSettings
import net.minecraft.block.Block
import net.minecraft.block.Material
import net.minecraft.item.BlockItem
import net.minecraft.registry.Registries
import net.minecraft.registry.Registry
import net.minecraft.sound.BlockSoundGroup
import net.minecraft.util.Identifier

object ModBlocks {
    // 基本的なブロック
    val EXAMPLE_BLOCK = Block(
        FabricBlockSettings.of(Material.METAL)
            .strength(4.0f)
            .requiresTool()
            .sounds(BlockSoundGroup.METAL)
    )
    
    fun register() {
        // ブロックの登録
        Registry.register(
            Registries.BLOCK,
            Identifier(ExampleMod.MOD_ID, "example_block"),
            EXAMPLE_BLOCK
        )
        
        // ブロックアイテムの登録
        Registry.register(
            Registries.ITEM,
            Identifier(ExampleMod.MOD_ID, "example_block"),
            BlockItem(EXAMPLE_BLOCK, FabricItemSettings())
        )
    }
}
```

### ブロックのテクスチャと言語設定の追加
1. `src/main/resources/assets/examplemod/textures/block/` ディレクトリを作成
2. `example_block.png` を追加
3. `src/main/resources/assets/examplemod/models/block/example_block.json` を作成:

```json
{
  "parent": "block/cube_all",
  "textures": {
    "all": "examplemod:block/example_block"
  }
}
```

4. `src/main/resources/assets/examplemod/models/item/example_block.json` を作成:

```json
{
  "parent": "examplemod:block/example_block"
}
```

5. `src/main/resources/assets/examplemod/blockstates/example_block.json` を作成:

```json
{
  "variants": {
    "": { "model": "examplemod:block/example_block" }
  }
}
```

6. 言語ファイル `en_us.json` に追加:

```json
{
  "block.examplemod.example_block": "Example Block",
  // 既存のアイテム翻訳もここに
}
```

## 7. クラフトレシピの追加

### レシピファイルの作成
`src/main/resources/data/examplemod/recipes/example_item.json` を作成:

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    " # ",
    "# #",
    " # "
  ],
  "key": {
    "#": {
      "item": "minecraft:iron_ingot"
    }
  },
  "result": {
    "item": "examplemod:example_item",
    "count": 2
  }
}
```

`src/main/resources/data/examplemod/recipes/example_block.json` を作成:

```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "###",
    "###",
    "###"
  ],
  "key": {
    "#": {
      "item": "examplemod:example_item"
    }
  },
  "result": {
    "item": "examplemod:example_block",
    "count": 1
  }
}
```

## 8. ルートテーブルの追加

### ブロックのルートテーブル
`src/main/resources/data/examplemod/loot_tables/blocks/example_block.json` を作成:

```json
{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "examplemod:example_block"
        }
      ],
      "conditions": [
        {
          "condition": "minecraft:survives_explosion"
        }
      ]
    }
  ]
}
```

## 9. MODのビルドと実行

### MODのテスト実行
1. IDEAのGradleタブを開く
2. Tasks > fabric > runClient をダブルクリック
3. Minecraftが起動し、MODが読み込まれる

### MODのビルド
1. IDEAのGradleタブを開く
2. Tasks > build > build をダブルクリック
3. ビルドが完了したら `build/libs/` ディレクトリにJARファイルが生成される
4. JARファイルのうち、名前に `-dev` や `-sources` が付いていないものが配布用

## 10. MODの配布

### 必要な依存関係の説明
ユーザー向けの説明書に以下の情報を明記:

1. Fabric Loader のインストールが必要
2. Fabric API のインストールが必要
3. Fabric Language Kotlin のインストールが必要
4. 対応Minecraftバージョン

### 配布プラットフォーム
以下のプラットフォームを使用して配布:

1. CurseForge
2. Modrinth
3. GitHub Releases

## 11. さらなる拡張のヒント

- アイテムグループ（クリエイティブタブ）の作成
- エンティティの追加
- カスタムブロックエンティティの作成
- カスタムGUIの実装
- クライアント/サーバー側の処理の分離
- Mixinを使ったバニラコードの拡張

このチュートリアルを基に、さらに複雑な機能を持つMODへと拡張していくことができます。 