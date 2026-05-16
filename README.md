# VRM4U PMX-Import 修正

これらの事項はバージョン更新毎に修正する必要がある.


---
## p01：ウェイト

### 対象：`Source/VRM4ULoader/Private/VrmSkeleton.cpp`

### 変更箇所**

- `snprintf(tmp, 512, ...)` の直後、`node->mName = tmp;` の直前に挿入.

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


---
## p02：透過テクスチャ有効化

### 対象：`Source/VRM4ULoader/Private/VrmConvertTexture.cpp`

### 変更箇所

* include追加

`#include "VrmUtil.h"` の直後に追加.

```cpp
#include "VrmBPFunctionLibrary.h"
```

- opacity処理追加

```cpp
// --- ここから ---
if (Options::Get().IsPMXModel()) {
    float opacity = 1.f;
    aiMat.Get(AI_MATKEY_OPACITY, opacity);
    UVrmBPFunctionLibrary::VRMChangeMaterialStaticSwitch(dm, TEXT("bUseDitherAlpha"), true);
    LocalScalarParameterSet(dm, TEXT("DitherAlpha"), opacity);
}
// --- ここまで ---

LocalMaterialFinishParam(dm);
```


---
## 確認
#### p01
- FName重複ボーンのウェイトが反映される

#### p02
- bUseDitherAlpha = trueが有効
- DitherAlphaにPMXの透過度が反映される
