# 花漾生活前端重新設計 — Flower Market 視覺識別

## Context

花卉電商「花漾生活」原本的前端視覺是一套完整、美學一致但偏「溫和花店範本」的設計（玫瑰粉 #C4727F + 鼠尾草綠 + 奶油色、Noto Serif/Sans TC、圓角卡片）。希望大膽重塑全新品牌識別，跳脫範本感。

採用 `/frontend-design` 技能搭配 Pencil MCP 的工作流程：先在 `.pen` 設計檔產出各頁設計稿與截圖交付確認，確認後再落地到 EJS + Tailwind 程式碼。

設計方向選定 **「鮮豔市集 Flower Market」**：靈感取自清晨花市的能量、手寫點貨牌與番路牌，避開常見 AI 預設樣貌（奶油襯線、黑底螢光、報紙細線）。後依使用者偏好，配色再微調為 **「蜜桃奶茶 Peachy Latte」**（女性向、溫柔氣質），保留市集手寫籤、直角邊框卡片等版面概念。

範圍涵蓋全部前台頁面：首頁、商品列表/詳情、購物車/結帳、登入/訂單。

---

## 設計系統

### 配色（蜜桃奶茶，最終採用）
| Token | Hex | 用途 |
|---|---|---|
| `coral`（蜜桃赭，主色） | `#C97D60` | 主要行動、價格、強調 |
| `coral-deep` | `#A8633F` | hover / 深化 |
| `marigold`（蜜桃粉） | `#F4B8A8` | 次要強調、標籤、手寫籤底色 |
| `ink`（可可棕） | `#5C4A42` | 文字主色、深色區塊背景、邊框 |
| `paper`（奶霜白） | `#FBF6F0` | 主背景 |
| `stem`（抹茶綠，點綴） | `#8FA888` | 庫存/狀態、少量點綴 |

### 字型
- **Display（標題/番路牌）**：`Archivo`（粗體壓縮感）
- **Body（內文）**：`Noto Sans TC`
- **Accent（手寫點貨牌）**：`Caveat`（手寫體，用於價格籤）

### 版面概念
- 直角/小圓角邊框卡片（`border-2 border-ink`，取代原本的柔陰影圓角卡片）
- 深色（`ink`）區塊作為節奏分隔（Hero、品牌故事、服務特色、訂單摘要欄）

### Signature
**手寫點貨牌價格籤**：商品價格以 `Caveat` 手寫體呈現，像花市攤位現場手寫的價格牌。

---

## 執行過程

### Phase A：Pencil 設計稿
1. 用 `set_variables` 建立配色/字型 token。
2. 建立 reusable components：`ProductCard`、`Button`、`PriceTag`、`Header`。
3. 逐頁設計稿（每頁拆成多個獨立 top-level frame，而非單一超高頁面 — 詳見下方踩坑記錄），逐段用 `get_screenshot`/`export_nodes` 驗證。
4. 完成後依使用者反饋，將全站配色由「鮮豔市集」改為「蜜桃奶茶」：解析所有節點的 fill/stroke，批次 `Update` 159 個節點完成色票置換。

### Phase B：落地 EJS + Tailwind
- `public/css/input.css`：重寫 `@theme` 配色 token，舊 token 名稱（`rose-primary` 等）保留作別名指向新色值，降低改動面、不影響後台頁面。
- `views/partials/head.ejs`：Google Fonts 改載入 Archivo + Caveat + Noto Sans TC。
- `views/partials/header.ejs`、`footer.ejs`：改為可可棕底、Archivo 粗體 logo。
- `views/pages/*.ejs`：index、product-detail、cart、checkout、login、orders、order-detail、404 全部套用新樣式（直角邊框卡片、手寫價格籤、Archivo 標題）。
- 執行 `npm run css:build` 重新編譯。

### 驗證
- Playwright 實機截圖逐頁比對（首頁、登入、商品詳情、購物車含商品狀態）。
- 加入購物車等動態互動功能確認正常運作。
- `npm run test`：原有 32 個 vitest 測試全數通過。

---

## 踩坑記錄（Pencil MCP 使用限制）

- **單一 frame 內容過高會不渲染**：把每頁拆成多個獨立的 top-level 區塊 frame（Header、Hero、Featured 等各自獨立），而非塞進一個大 frame。
- **`fill`/`stroke` 用 `$變數` 引用在背景色上偶不生效**：圖形元素的顏色改用直接 hex 值更穩定，文字 fill 用變數沒問題。
- **`get_screenshot` 有間歇性渲染失敗**（資料結構確認正確但截圖空白），重試或改用 `export_nodes` 可繞過；`export_nodes` 偶爾會報 "wrong .pen file" 錯誤，需切換回 `get_screenshot` 重試。
- **最外層用 `justifyContent/alignItems: center` 置中容易渲染異常**：改用 `layout: "none"` + `layoutPosition: "absolute"` 明確定位更穩定。
- **vertical/horizontal layout 中以 `fit_content` 自動撐高的子 frame 偶爾會重疊/塌陷**：給子層明確 `height` 可解決。

## 關鍵檔案
- `public/css/input.css`、`views/partials/head.ejs`、`header.ejs`、`footer.ejs`
- `views/pages/index.ejs`、`product-detail.ejs`、`cart.ejs`、`checkout.ejs`、`login.ejs`、`orders.ejs`、`order-detail.ejs`、`404.ejs`
- 後台頁面（`admin/*`）不在本次範圍內，舊配色 token 別名機制確保其不受影響。
