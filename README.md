# deps-test

This repo reproduces a pnpm + prefab Android build failure when any package is patched and `nodeLinker: hoisted` is not set in `pnpm-workspace.yaml`.

## Bug reproduction

The patch applied to `react-native-reanimated` causes its package path in `node_modules/.pnpm` to include the patch hash (e.g. `react-native-reanimated@4.2.1_patch_hash=...`). When `nodeLinker: hoisted` is absent from `pnpm-workspace.yaml`, the native module is not hoisted correctly and the [prefab C++ build system fails](https://github.com/google/prefab/issues/187) because it receives the raw pnpm path as an option, which it cannot parse.

**Expected error:**
```
Error: no such option /Users/.../node_modules/.pnpm/react-native-reanimated@4.2.1_patch_hash=...
com.android.ide.common.process.ProcessException: C++ build system [prefab] failed while executing:
  /Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home/bin/java \
  ...
```

### Steps to reproduce

1. Remove `nodeLinker: hoisted` from `pnpm-workspace.yaml` (or ensure it is absent)
2. Install dependencies:
   ```bash
   pnpm install
   ```
3. Run the Android debug build:
   ```bash
   npx expo run:android --variant debug
   ```

The build will fail with the prefab error above.
