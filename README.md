# botanoalarm-config

**Botano / 我的鬧鐘** 這支 App 對外公開的東西都放這裡。三個檔案：

| 檔案 | 是什麼 | 什麼時候要動它 |
|---|---|---|
| **[`holiday_config.json`](holiday_config.json)** | App 去哪裡抓台灣的放假資料 | 假日標記出問題時（見下） |
| **[`privacy.html`](privacy.html)** | 隱私權政策網頁版（繁中） | App 的資料行為改變時 |
| **[`privacy-en.html`](privacy-en.html)** | 隱私權政策網頁版（英文） | 同上，**兩份要一起改** |

> ⚠️ **這個 repo 必須永遠公開、而且不能改名或刪除。**
> `holiday_config.json` 的網址寫在 App 裡，兩份 `privacy` 的網址填在
> Play Console。全都是外面指進來的，動了就是斷掉。

---

## 隱私權政策：改它之前先看這裡

**這兩份都是副本，正本在 App 的原始碼裡**（`doc/privacy_policy.md` 與
`doc/privacy_policy_en.md`）。

**中英文不是互相翻譯，是兩份各自成立的文件。** 例如政府假日那一節：中文版直接
介紹功能，英文版寫的是「這是台灣專屬的功能、在海外預設關閉」——因為對海外
使用者來說那件事根本不會發生。改動時要各自看語意，不要對照著翻。

原始碼那個 repo 是私有的，開不了 GitHub Pages，所以隱私權政策才需要在這裡放一份。

**要改就兩邊一起改。** 只改一邊的話，商店上寫的與 App 裡（設定 →「關於」）顯示的會是兩套說法——那不只是不一致，在法律上是個問題，Play 審核也會抓。

正本改完之後，用專案裡的轉檔腳本重新產生這一頁，**不要手改 HTML**（手改的話，下次跑腳本就會被蓋掉）。在 App 的專案目錄下執行：

```
python tool/privacy_to_html.py doc/privacy_policy.md    ../botanoalarm-config/privacy.html
python tool/privacy_to_html.py doc/privacy_policy_en.md ../botanoalarm-config/privacy-en.html
```

最後那個路徑換成你這個 repo 的實際位置。**兩行都要跑。**

已知還有一次改動是排定要做的：**內購上線後**，要回頭確認第四節寫的 RevenueCat 行為與實際串接一致。

網址。**Play Console 的隱私權政策網址是每個商店語言各自填一個**，別填錯：

| 商店語言 | 要填的網址 |
|---|---|
| 繁體中文 | <https://tnth.github.io/botanoalarm-config/privacy.html> |
| 英文 | <https://tnth.github.io/botanoalarm-config/privacy-en.html> |

---

## holiday_config.json：這是什麼

App 需要知道台灣哪幾天放假（國定假日、補班日），才能讓「依行事曆」的鬧鐘在該休息的日子不要響。那份資料不是我們自己維護的，是去抓別人的公開資料。

**這個檔案就是在告訴 App「去哪裡抓」。**

它放在 GitHub 而不是寫死在 App 裡，是為了一件事：**資料來源哪天壞掉時，不用重發 App 就能修**。

改一個檔案，所有使用者的手機在 30 天內自己接上新的來源。沒有這個機制的話，唯一的修法是發一版新 App、等 Google 審核、再等使用者更新——而不更新的人永遠修不好。

（改成私有的話 App 會抓不到——不會壞掉，會退回內建那份，但這個機制就等於不存在了。）

---

## 什麼時候需要改

平常不用管。下面兩種情況才要動它。

### 情況一：假日資料抓不到了

**症狀**：使用者回報「國定假日沒有自動標成休假」「補班日沒有標成上班」，而且不只一個人。

**原因**：`holidaySources` 裡的網址失效了。目前用的是 [ruyut/TaiwanCalendar](https://github.com/ruyut/TaiwanCalendar)，那是別人的 repo——他可能改名、搬家、改資料格式，或單純不做了。

**怎麼確認**：把網址裡的 `{year}` 換成今年，直接用瀏覽器打開看看：

```
https://cdn.jsdelivr.net/gh/ruyut/TaiwanCalendar/data/2027.json
```

- 看到一大串 JSON → 來源正常，問題在別的地方
- 404 或空白 → 來源掛了，要換

**怎麼修**：見下面「換掉資料來源」。

### 情況二：某個紀念日被誤標成放假

**症狀**：使用者回報某天明明要上班，App 卻自動標成休假。

**原因**：政府資料把「紀念日」和「放假日」混在一起。例如**軍人節（9/3）**在資料裡是節日，但只有軍人放假，一般人照常上班。

**怎麼修**：把那天加進 `holidayExclusions`，見下面「排除某個紀念日」。

---

## 怎麼改

### 換掉資料來源

改 `holidaySources` 這個清單。

```json
"holidaySources": [
  "https://cdn.jsdelivr.net/gh/ruyut/TaiwanCalendar/data/{year}.json",
  "https://raw.githubusercontent.com/ruyut/TaiwanCalendar/main/data/{year}.json"
]
```

**規則：**

| 規則 | 說明 |
|---|---|
| `{year}` 不要拿掉 | App 會自動代入年份（`{year}` → `2027`）。**寫死年份的話每年都要回來改一次**，這個機制就白做了 |
| 由上往下試 | 第一個抓得到就用它，抓不到才換下一個。所以最可靠的放最前面 |
| 至少要有一個 | 清單空了的話整份設定檔會被當成無效，App 直接改用內建的那份 |
| 可以放很多個 | 多寫幾個不同來源當備援是好事 |

**新來源的資料格式必須長這樣**（一個陣列，每天一筆）：

```json
[
  { "date": "20260101", "week": "四", "isHoliday": true, "description": "開國紀念日" },
  { "date": "20260102", "week": "五", "isHoliday": false, "description": "" }
]
```

`date`、`isHoliday`、`description` 這三個欄位是必要的（`week` 用不到）。找不到同格式的來源時，就不是改設定檔能解決的了，得改 App。

### 排除某個紀念日

加進 `holidayExclusions`：

```json
"holidayExclusions": [
  { "date": "09-03", "note": "軍人節，只有軍人放假，不是普遍放假日" }
]
```

**規則：**

| 規則 | 說明 |
|---|---|
| 格式是 `MM-DD` | **不含年份**。軍人節每年都是 9/3，寫一次就好，不用每年加一筆 |
| 一定要兩位數 | `09-03` ✅　`9-3` ❌（格式錯的那一筆會被安靜略過） |
| `note` 只給你自己看 | 不會顯示給使用者。寫清楚「為什麼加這筆」，未來的你會需要 |
| 只影響「自動放假」 | 使用者自己手動標的那天不受影響 |

---

## 改完之後

### 1. 檢查 JSON 有沒有寫壞

貼到 <https://jsonlint.com> 按一下就知道。少一個逗號、多一個逗號都會讓整份檔案失效。

### 2. 直接 commit 到 `main`

不需要開分支或 PR，這是一個人維護的設定檔。

### 3. 確認 App 抓得到

用瀏覽器打開這個網址，看到的內容應該和你剛改的一樣：

```
https://cdn.jsdelivr.net/gh/tnth/botanoalarm-config@main/holiday_config.json
```

**這正是 App 去抓的那個網址。** 打不開的話，使用者也抓不到。

### 4. 等它傳開

| 環節 | 延遲 |
|---|---|
| jsDelivr 的快取 | 最多約 12 小時 |
| 使用者的手機 | 距上次成功同步滿 30 天後的下一次開 App |

所以**最慢一個多月**才會全部更新完。急不得，這個機制本來就是給「反正也不急」的問題用的。

> 想跳過 jsDelivr 的 12 小時快取來確認自己改對了，可以看這個立即生效的網址（內容一樣）：
> `https://raw.githubusercontent.com/tnth/botanoalarm-config/main/holiday_config.json`

---

## 寫壞了會怎樣

**不會有人的鬧鐘因此不響。** 這件事在設計上就被隔開了——假日資料只影響「哪一天要不要響」，鬧鐘本身的排程與響鈴完全不碰這裡。

而且每一層都有退路：

| 出了什麼事 | App 的反應 |
|---|---|
| 這個檔案抓不到 | 改用打包在 App 裡的那份設定（內容就是現在這份的副本） |
| JSON 語法壞掉 | 同上，整份丟掉、改用內建的 |
| `holidaySources` 空了 | 同上 |
| 某一筆排除項目格式錯 | **只丟掉那一筆**，其餘照常生效 |
| 資料來源網址全部抓不到 | 這次同步失敗，保留上次抓到的資料，30 天後再試 |

也就是說，最壞的情況是「退回你發 App 那天的狀態」，不是壞掉。

---

## 用不到但值得知道的

- App 每 30 天才同步一次，不是每次開啟都抓。改了之後不會立刻看到效果是正常的。
- 使用者可以在設定裡關掉「政府假日」，關掉的人完全不受這個檔案影響。
- 同步失敗是安靜的，不會跳任何訊息給使用者——他們只會看到假日標記停在上次的狀態。
- 這個 repo 沒有任何機密，公開它沒有損失。App 的原始碼在另一個私有 repo。
