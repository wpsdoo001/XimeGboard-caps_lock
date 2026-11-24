# XimeGboard-caps_lock
基于XimeGboard优化个人使用,增加点击shift按键,开启大写字母锁定功能
原项目链接:[(https://github.com/Murchey/Xime-Gboardlike)]
# Xime 输入法（Murchey/Xime-Gboardlike）· 新增「锁定大写」

## 功能行为

设置 → 键盘偏好 →「按键」分组，在「字母大小写 / 跟随 Shift 键切换」之后新增 **锁定大写** 开关（默认关闭）。

- 打开：单击 Shift 即锁定英文大写（键面箭头由空心变实心并在下方出现横条），**再次单击解除**。
- 关闭：完全保持原有行为——单击临时大写一个字符、双击锁定大写、连点解除。
- 生效范围：26 键主键盘、左右分体 Shift、数字/符号面板上的 shift 键，以及配置为 `toggle_shift` 的自定义键位。
- 状态随键盘重置逻辑（切换中英文方案等）一致复用原有规则，未新增额外复位行为。

## 代码改动（4 个文件，约 40 行，均为增量）

1. `settings/SettingsPreferences.kt`：新增 `KEY_SHIFT_CAPS_LOCK`（`shift_caps_lock`）与 `isShiftCapsLockEnabled()`，默认 false。
2. `ui/settings/LayoutDisplaySettingsSpecs.kt`：以声明式设置项 `SettingSpec.Switch` 加入「键盘偏好」页，键名与偏好共用同一份 SharedPreferences。
3. `viewmodel/KeyboardViewModel.kt`：
   - `singleTapShift()`：开关开启时改为 CAPS ⇄ OFF 切换；
   - `toggleShift()`：开关开启时委托给 `singleTapShift()`，让 `toggle_shift` 键位同样生效；
   - `doubleTapShift()`：开关开启时，若距上次单击不足 400ms 则忽略（避免手速快的第二击把刚锁定的大写又解除），专属 caps 键（走 `shift_caps`）行为不变。
4. `res/values/strings.xml`、`res/values-zh-rHK/strings.xml`：新增「锁定大写」「鎖定大寫」文案与说明。

补丁文件：`caps-lock.patch`（相对仓库根，只含以上功能改动，不含本地打包配置）。

## APK 说明

- 文件：`XimeGboard-1.2.2-arm64-capslock.apk`，119 MB
- 包名 `com.ximegboard`，versionName `1.2.2`，versionCode `20261024`，minSdk 28，targetSdk 35
- 仅含 **arm64-v8a**（目前绝大多数手机）；签名使用 Android Debug 证书，可直接安装
- SHA-256：`82c201725329cb728529dcdc6dd7152b8e2ffd8d1548bcf9ead2bf6ce61d2b6d`

### 构建方式说明

本次只改 Kotlin/Compose 层，未触碰任何 native 代码：

- native 库（`librime_jni` / `onnx` / `quickjs` / `asr` / `handwriting` 等）与 rime 词典资源直接取自官方 V1.2.2 发布包，已逐一核对 Kotlin 侧 106 个 `external fun` 与 `.so` 导出符号一一对应；
- 因此本地打包时跳过了 NDK 编译（`externalNativeBuild` 与 onnx/knf 下载任务未参与），其余 Gradle 配置与原仓库一致；
- 官方发布包没有包含新增的「设备互联」native 库（`libxime_p2p_jni.so`），当前代码在缺失时会自动降级、不影响其他功能；其余功能与官方 1.2.2 版本一致。

### 安装提示

APK 使用调试证书签名，与官方发布版签名不同。如果手机上已安装官方版 Xime，请先卸载再安装本包，否则会提示签名冲突。

<img width="108" height="237" alt="Screenshot_2026-11-24-19-14-25-93_8dbdd6c41b3dfb44b35a197727925186" src="https://github.com/user-attachments/assets/0f8dbeda-d703-4b6f-9bc7-712752d2dd5d" />
