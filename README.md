# qianey-legal

钱伲（Qianey）的公开法务/支持页，托管在 GitHub Pages 上，供 App Store Connect 填写两个必填 URL。

**这个仓库会公开可访问，不要往这里放任何内部文档。**
钱伲主仓库（私有的 `DangYangJiRan`）里也有一份 `docs/` 目录，里面是**内部**同步 Q&A 和待修复问题清单——
那些**不能**进这个仓库，两个目录不要互相复制。

| 文件 | 对应的 App Store Connect 字段 |
|---|---|
| `privacy.html` | **隐私政策 URL**（送审阻断项，所有 App 必填） |
| `support.html` | **支持 URL** |
| `index.html` | 站点首页，给从根路径点进来的人（含 App Store 审核员）一个入口 |

## ⚠️ 发布前必须替换的占位符

```bash
grep -rn "【" *.html     # 必须无输出才能发布
```

- `【支持邮箱】` —— `privacy.html` 和 `support.html` 各两处（中英各一）
- `【开发者名称】` —— `privacy.html` 顶部生效日期旁，中英各一处。
  填 App Store 上的开发者/卖方名称，与商店信息保持一致

## 首次发布

```bash
cd ~/Documents/AI/qianey-legal
git init
git add .
git commit -m "docs: 钱伲隐私政策与支持页"
git branch -M main

# 先在 GitHub 上建一个【公开】仓库，名字建议 qianey-legal（不要用钱伲主仓库）
git remote add origin git@github.com:yuxiao345/qianey-legal.git
git push -u origin main
```

然后到该仓库 **Settings → Pages**：

- **Source**：`Deploy from a branch`
- **Branch**：`main`，文件夹 `/ (root)`
- Save，等 1–2 分钟

最终 URL：

- 首页 `https://yuxiao345.github.io/qianey-legal/`
- 隐私政策 `https://yuxiao345.github.io/qianey-legal/privacy.html`
- 支持 `https://yuxiao345.github.io/qianey-legal/support.html`

把后两个填进 App Store Connect 的「隐私政策 URL」与「支持 URL」。两个 URL 在中文和英文本地化里都可以填同一个。

## 后续修改

改完直接 `git push`，URL 不变，**不需要重新送审**。

但要注意：**内容与 App 实际行为不符属于 Guideline 2.3 虚假元数据。**
改了 App 的数据处理方式（新增 SDK、新增网络请求、新增权限、新增数据类型），
必须回头改这里的隐私政策。下面是每条主张的依据，改之前先核对，别写成不实的。

## 页面内容的依据（对着钱伲主仓库代码核过）

| 主张 | 依据 |
|---|---|
| 不注册、不收集手机号/邮箱/身份信息 | 全项目无账号体系、无登录代码 |
| 零第三方依赖、无广告/分析/追踪 SDK | 项目无任何包依赖；`PrivacyInfo.xcprivacy` 里 `NSPrivacyTracking = false` |
| 数据在我们读不到的私有 iCloud 库 | `SyncServiceImpl` 只用 `privateCloudDatabase`；共享走 `sharedCloudDatabase`；**全项目无 `publicCloudDatabase`**，开发者看不到任何用户记录 |
| 唯一的外部请求是汇率 | `CurrencyServiceImpl` / `ExchangeRateServiceImpl` 请求 `https://api.frankfurter.app`，由 `AddEditTransactionView:2355`（iOS）、`MacAddTransactionSheet:1264`（Mac）在**币种与本位币不同**时触发。全项目再无其他 `URLSession` 调用 |
| 照片只在用户主动选择时才读，不申请图库权限 | 交易的「附件」用 `PhotosPicker`（进程外选择器）；`Info.plist` 里**没有** `NSPhotoLibraryUsageDescription` |
| 生物识别 App 拿不到数据 | `BiometricAuth.swift` 用 `LAContext.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics)`，仅返回成功/失败 |
| App 锁忘记可重装解决 | `appLockEnabled` 是 `@AppStorage`（UserDefaults），删除 App 即清除；账目在 iCloud，重装后同步回来 |
| 导 CSV / JSON | `ExportServiceImpl` |
| iOS 18.0+、iPhone 与 iPad | `IPHONEOS_DEPLOYMENT_TARGET = 18.0`、`TARGETED_DEVICE_FAMILY = 1,2` |

### 两条措辞红线

- **别写「数据不上传到服务器」。** 本 App 走 CloudKit，数据会同步进用户自己的 iCloud，
  那句话是假的。正确说法是「存在你自己的 iCloud 私有空间，我们无法读取」。
- **别写「本 App 不具备导入能力」。** 信用卡账单 CSV 导入是随正式版发布的
  （`CreditCardReconciliationView.swift:310`，未包 `#if DEBUG`）。
  OCR 识别测试是 `#if DEBUG` 专有，正式包不含，所以「不做截图识别记账」成立。
