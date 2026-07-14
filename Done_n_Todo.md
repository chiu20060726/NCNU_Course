# Done
1. 補足1131~1151歷年課程資料
2. 修正getData.py

# 修正大綱(by AI)
1. 根本問題修復：課程資料抓取邏輯
舊邏輯（已刪除）：送出 srh[...] 搜尋條件後，直接解析回應 HTML 裡 setDataTables(...) 這段 JS 呼叫字串，把裡面的參數當 JSON 解析出來當作課程資料。

問題：學校網站改版後，這段 JS 呼叫裡的 "data":[] 永遠是空的初始化殼子——真正的課程資料是頁面載入後，瀏覽器另外用 DataTables 的 server-side AJAX 協議發一次請求才拿到的。所以舊邏輯不管抓哪個學期都是空的，跟「學期有沒有開課」無關。

新邏輯（getData.py:69-108）：

第一段 srh[...] 搜尋 POST 保留，用來把篩選條件（哪個學年、哪個學期）存進 session
新增 buildDataTablesPayload()：組出 DataTables 標準格式的請求參數（draw、start、length、columns[i][data/name/searchable/orderable/search...]）
先用 length=1 探一次 recordsTotal（總筆數），再用剛好的 length 送第二次請求一次拿滿全部資料，改用 X-Requested-With + X-CSRF-TOKEN header 標示為 AJAX 請求
直接解析回傳的 JSON（result['data']），取代原本脆弱的字串切割解析
2. 資料型態防呆（getData.py:120-126）
新版 API 對缺值欄位回傳 JSON null，不是空字串 ""。原本 course['SemCourseTime'] != "" 這種判斷式碰到 null 會在 .replace() 時當掉，改成用 truthy 判斷（if course['SemCourseTime']）同時涵蓋 None 和 "" 兩種情況；ClassRoom、Memo 同樣處理。

# to-do
1. 更新VUE的文字(/js沒改到，我就加進.gitignore了)
2. 更新網站