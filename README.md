# VRM4U PMX パッチ適用手順書

アップデート後に毎回適用が必要なパッチ一覧。

---

## パッチ①：ウエイト

**対象ファイル**
`Source/VRM4ULoader/Private/VrmSkeleton.cpp`

**変更箇所**
`snprintf(tmp, 512, ...)` の直後、`node->mName = tmp;` の直前に挿入。

```cpp
// --- ここから挿入 ---
for (uint32 m = 0; m < scene->mNumMeshes; ++m) {
    auto* aiM = scene->mMeshes[m];
    for (uint32 b = 0; b < aiM->mNumBones; ++b) {
        auto* aiB = aiM->mBones[b];
        if (aiB->mNode == node) {
            aiB->mName = tmp;
        }
    }
}
// --- ここまで挿入 ---

node->mName = tmp;
```

**目的**：同名ボーンのリネーム時、そのノードを参照するボーンの `mName` も同期してリネームする。

---

## パッチ②：透過（マテリアル）

**対象ファイル**
`Source/VRM4ULoader/Private/VrmConvertTexture.cpp`

### ② -A：include 追加

`#include "VrmUtil.h"` の直後に追加。

```cpp
#include "VrmBPFunctionLibrary.h"
```

### ② -B：opacity 処理追加

gltfテクスチャループ（`for (uint32_t t = 0; ...`）終了の直後、`LocalMaterialFinishParam(dm);` の直前に挿入。

```cpp
// --- ここから挿入 ---
if (Options::Get().IsPMXModel()) {
    float opacity = 1.f;
    aiMat.Get(AI_MATKEY_OPACITY, opacity);
    UVrmBPFunctionLibrary::VRMChangeMaterialStaticSwitch(dm, TEXT("bUseDitherAlpha"), true);
    LocalScalarParameterSet(dm, TEXT("DitherAlpha"), opacity);
}
// --- ここまで挿入 ---

LocalMaterialFinishParam(dm);
```

**目的**：PMXモデルの各マテリアルの不透明度（opacity）をディザアルファとしてマテリアルに反映する。

---

## パッチ③：透過（モーフターゲット）

**対象ファイル**
`Source/VRM4ULoader/Private/VrmConvertMorphTarget.cpp`

**変更箇所**（`UE_VERSION_OLDER_THAN(5,0,0)` 分岐内）

変更前：
```cpp
#if UE_VERSION_OLDER_THAN(5,0,0)
    if (mt->MorphLODModels.IsValidIndex(0) == false) {
        mt->MorphLODModels.Add(MorphLODModel);
    } else {
        mt->MorphLODModels[0] = MorphLODModel;
    }
#else
    if (mt->GetMorphLODModels().IsValidIndex(0) == false) {
        mt->GetMorphLODModels().Add(MorphLODModel);
    } else {
        mt->GetMorphLODModels()[0] = MorphLODModel;
    }
#endif
```

変更後：
```cpp
#if UE_VERSION_OLDER_THAN(5,0,0)
    mt->MorphLODModels.Add(MorphLODModel);
#else
    mt->GetMorphLODModels().Add(MorphLODModel);
#endif
```

**目的**：MorphLODModels への追加時の `IsValidIndex` チェックと上書き分岐を削除し、常に `Add()` する。

---

## 確認方法

パッチ適用後、以下のキーワードで検索して存在を確認。

| パッチ | 確認キーワード |
|--------|----------------|
| ① | `aiB->mNode == node` |
| ②-A | `VrmBPFunctionLibrary.h` |
| ②-B | `bUseDitherAlpha` |
| ③ | `MorphLODModels.Add(MorphLODModel)` （IsValidIndex が**ない**こと） |
