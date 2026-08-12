【商業簡介網站｜8 項功能升級版】

這一版是以你上傳的「商業簡介網站_登入修正版(1).zip」為基礎修改。

已加入：
1. 首頁最新消息只顯示最新 3 則，並有「更多消息」頁面。
2. 訪客使用管理員建立的 Supabase Auth 帳號登入。
3. 公告刪除改成檢查 HTTP 回應與 RLS，並加入編輯功能。
4. 商品頁面＋後台新增／編輯／上架／下架／刪除商品。
5. 公告支援置頂與預定發布時間。
6. 公告分類：最新消息／活動／蛋仔／Minecraft伺服器更新。
7. 管理員可替訪客重設密碼，訪客沒有自行重設入口。
8. 客服：訪客送出訊息，後台可查看並回覆。

【第一次設定】
1. 把你原本可用的 config.js 放回本資料夾，覆蓋目前範例檔。
2. 在 Supabase SQL Editor 執行 schema_upgrade.sql。
3. 執行 Edge Functions：
   supabase functions deploy create-visitor
   supabase functions deploy reset-visitor-password
4. 兩個 Edge Function 都需要 Supabase 自動提供的 SUPABASE_URL、SUPABASE_ANON_KEY、SUPABASE_SERVICE_ROLE_KEY 環境變數。
5. schema_upgrade.sql 會把當下已存在的 Auth 使用者建立成 admin。若有多個既有使用者，請手動把真正管理員設成 admin，例如：
   update public.profiles p set role='admin' from auth.users u where p.id=u.id and u.email='你的管理員Email';
6. 新建立的訪客會由 Edge Function 建立為 visitor。

【重要安全提醒】
- 不要把 service_role key 放進 config.js 或任何前端 JS。
- config.js 只放 Supabase URL 與 anon/publishable key。
- 訪客密碼由 Supabase Auth 管理，不會存進 visitor_accounts。
- 如果你在 Supabase 啟用了 Email 驗證，Edge Function 建立訪客時會直接 email_confirm=true；這符合「由管理員建立帳號」的需求。

【本機測試】
可用任何靜態網站伺服器開啟資料夾，例如：
python -m http.server 8000
然後開 http://127.0.0.1:8000/index.html

沒有你的實際 Supabase URL/Key 與資料庫環境時，我可以測試前端檔案、JS 語法、頁面載入與結構，但不能假裝已測試你的真實登入、RLS、Edge Function 和資料庫。


【本次修正版：訪客帳號 CORS】
如果建立訪客時出現「Failed to fetch」或 create-visitor 的 CORS 錯誤，
請務必把這兩個 Function 重新部署，不能只更新前端 ZIP：

supabase functions deploy create-visitor
supabase functions deploy reset-visitor-password

新版 Function 已處理 OPTIONS 預檢請求，並加入：
Access-Control-Allow-Methods: POST, OPTIONS
Access-Control-Allow-Headers: authorization, x-client-info, apikey, content-type

部署完成後重新整理 GitHub Pages，再測試「後台 → 訪客帳號 → 建立訪客」。

注意：如果你是直接在 Supabase Dashboard 編輯 Edge Function，
請把本 ZIP 裡對應的 index.ts 完整貼入並重新 Deploy。
