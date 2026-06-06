# CLAUDE.md — 維護與協作指南

本檔給在此 repo 工作的 AI 助手（與維護者）閱讀，說明**這份憲章是什麼、如何維護、以及變更前的討論流程**。
本檔只談「維護方式」；憲章內容本身以 [`constitution.md`](constitution.md) 及各延伸檔為準。

---

## 1. 這個 repo 是什麼

團隊的 **Spec-Driven Development（SDD）憲章**，採「**核心原則 + 服務風險分級 + 應用/基礎設施延伸**」結構，並以**前綴式條款編號**組織。對外採用方式見 [`README.md`](README.md)（三階段循序導入，預設第一階段）。

### 檔案地圖

| 檔案 | 前綴 | 內容 |
|------|------|------|
| [`constitution.md`](constitution.md) | CP / TR / GV | 核心原則、分級要求、治理、服務風險分級、附錄 A（環境變數）、附錄 B（分級適用矩陣）|
| [`constitution-api.md`](constitution-api.md) | API | API 服務應用面（REST / WebSocket）|
| [`constitution-frontend.md`](constitution-frontend.md) | FE | 前端應用面 |
| [`constitution-shell.md`](constitution-shell.md) | SH | Shell Script 應用面 |
| [`constitution-microservice.md`](constitution-microservice.md) | SVC | 跨服務治理（契約/事件/追蹤/韌性/資料邊界/並行/生命週期）|
| [`constitution-deployment.md`](constitution-deployment.md) | DEP | 部署與基礎設施（容器/Compose/第三方/DB/儲存/代理）|
| [`tech-standards.md`](tech-standards.md) | —（獨立版本）| 技術選型建議（非強制 SHOULD）|
| [`README.md`](README.md) | — | 團隊三階段導入指南 |

### 三個邊界（搬移條款時務必遵守）
- **應用延伸（API/FE/SH）**：只規範該應用本身的設計與行為。
- **SVC**：跨服務治理。
- **DEP**：部署與基礎設施（橫切，適用所有自建可部署服務）。
> 凡涉及「服務部署」或「跨服務溝通治理」的條款，**不可**散落在應用延伸，應分別歸入 DEP 或 SVC。

---

## 2. 維護慣例（硬規則）

1. **前綴式編號**：條款 ID 一律 `§<PREFIX>-<n>`（CP/TR/GV/API/FE/SH/SVC/DEP）。**不使用全域章號**（不要寫「第七章」）。新增條款時往該前綴後遞增。
2. **交叉引用**：跨檔引用直接用 ID（如「見 §SVC-2」「見 §DEP-7」）。改動編號時，**MUST** 同步更新所有引用，並跑第 3 節的驗證。
3. **版本同步**：六份憲章檔（constitution + 5 延伸）共用同一版本號；`tech-standards.md` 獨立版本。版本依 §GV-1（SemVer）：
   - **MAJOR**：移除/重新定義原則，或**重編條款識別碼**（造成既有引用失效）。
   - **MINOR**：新增條款或實質擴充。
   - **PATCH**：文字釐清、錯字。
4. **每個延伸檔開頭**保留：版本行、`§<PREFIX>-` 說明、**邊界聲明**、分級適用註記。
5. **勿重新引入**已被精簡掉的東西：需求溯源註記（「對應使用者需求 #X」）、重複的分層表附錄、各檔版本歷史表。
6. **附錄 B 矩陣**欄位為服務風險分級（Critical/Standard/Internal/Experimental）；核心原則（CP）不列入矩陣（皆 MUST、不可豁免）。
7. **README 的「目前導入階段」**為對外採用起點，預設第一階段；調整階段政策時一併檢視 README。

---

## 3. 一致性驗證（每次改動編號/引用後必跑）

```bash
cd "<repo>"

# (a) 不應有殘留的舊式章節號或舊引用
grep -nE "§[0-9]+\.[0-9]+|第[一二三四五六七八]章|ms §|api §[0-9]|frontend §[0-9]|shell §[0-9]|deploy §[0-9]|核心 §[0-9]" *.md || echo "OK: 無殘留"

# (b) 所有被引用的前綴 ID 都必須有對應章節定義（輸出無 MISSING 即通過）
defined=$(grep -hoE "^### §(CP|TR|GV|API|FE|SH|SVC|DEP)-[0-9]+" *.md | sed 's/^### //' | sort -u)
used=$(grep -hoE "§(CP|TR|GV|API|FE|SH|SVC|DEP)-[0-9]+" *.md | sort -u)
for u in $used; do echo "$defined" | grep -qx "$u" || echo "MISSING: $u"; done; echo "check done"

# (c) 版本一致性（六份憲章應同版號）
grep -hE "版本：" *.md
```

---

## 4. 變更討論流程（重要）

任何**非瑣碎**的憲章調整，遵循「**草案 → 第二方 AI 評估 → 與使用者討論確認 → 才修改 → 驗證**」：

1. **整理草案 / 動機**：說明要改什麼、為什麼、影響哪些檔案與引用。
2. **取得第二方 AI 評估**：用**另一個 AI agent** 做獨立 review，提供不同視角（避免單一模型盲點）。
   - 目前管道：**本地 copilot CLI + GPT-5.5**（見第 5 節指令）。
   - **可替換**為其他工具（如 codex / 其他模型）；只要能讀檔並給出獨立評估即可。
3. **彙整評估重點，與使用者討論**：把第二方意見摘要化（同意/不同意/取捨），**明確標示這是外部 AI 的建議，供權衡**，並提出可選範圍讓使用者選擇。
4. **使用者確認範圍後才修改**：**MUST NOT** 在未確認前逕行大幅重構。對應 §CP-5（變更需經審查、人為核准）。
5. **修改後**：跑第 3 節驗證 → 視變更幅度更新版本（§GV-1）→ 更新本檔/README/memory（若結構或政策改變）。

> 原則：AI（本助手或第二方）負責**產出與評估**，**決策與確認由使用者**。這正是憲章 §CP-5 / §GV-3 的精神。

---

## 5. 第二方 AI 評估指令（copilot CLI + GPT-5.5）

```bash
copilot --model gpt-5.5 --allow-all-tools -p "<在此放入評估問題；請對方閱讀本目錄下的 constitution*.md、tech-standards.md、README.md 後，以資深架構師角度用繁體中文條列評估與可執行建議>" 2>/dev/null
```

注意事項：
- **沙箱**：copilot 需寫入 `~/.copilot`，在沙箱中會出現 `EPERM/EACCES`。需以**停用沙箱**方式執行該指令。
- **雜訊**：啟動時的 `failed to copy trust settings of system certificate` 屬無害 stderr，可用 `2>/dev/null` 過濾。
- **模型可換**：`--model` 可改其他可用模型；或改用 codex 等其他 CLI——流程不變，重點是取得**獨立第二意見**。
- 評估問題建議聚焦：是否過於複雜/嚴苛、結構是否合理、條款歸屬（應用面 vs SVC vs DEP）、缺漏與落地建議。

---

## 6. 演進脈絡（快速背景）

v1 單檔 → v1.2 多檔拆分（token 效率）→ v2 把過嚴的 MUST 再平衡為 SHOULD、框架選型移到 tech-standards → v3 核心原則+專案分級+延伸 → v4 專案分級改**服務風險分級**、新增 SVC（微服務）與 DEP（部署）延伸、API 改協定中立 → v5 **前綴式編號（CP/TR/GV/API/FE/SH/SVC/DEP）** + 三邊界明確化。詳細決策記錄於助手 memory。
