import requests
import matplotlib.pyplot as plt
import datetime

# ====== 設定你的 RapidAPI Key ======
API_KEY = "你的RapidAPI_KEY"

# ====== 股票代碼 ======
symbol = "AAPL"

# ====== Rapid API 設定 ======
url = f"https://financialmodelingprep.p.rapidapi.com/v3/ratios/{symbol}?limit=20&apikey={API_KEY}"

headers = {
    "X-RapidAPI-Key": API_KEY,
    "X-RapidAPI-Host": "financial-modeling-prep.p.rapidapi.com"
}

# ====== 發送 API 請求 ======
response = requests.get(url, headers=headers)
data = response.json()

# ====== 整理資料 ======
dates = []
pe_ratios = []
ps_ratios = []

for entry in data:
    dates.append(datetime.datetime.strptime(entry['date'], "%Y-%m-%d").date())
    pe_ratios.append(entry['priceEarningsRatio'])
    ps_ratios.append(entry['priceToSalesRatio'])

# 只取最近5年的資料（每年一筆）
dates = dates[:5]
pe_ratios = pe_ratios[:5]
ps_ratios = ps_ratios[:5]

# ====== 計算平均 PE / PS ======
average_pe = sum(pe_ratios) / len(pe_ratios)
average_ps = sum(ps_ratios) / len(ps_ratios)

# ====== 畫出 PE Ratio 走勢 ======
plt.figure(figsize=(12, 6))
plt.plot(dates, pe_ratios, marker='o', label='PE Ratio')
plt.axhline(average_pe, color='red', linestyle='--', label=f'平均 PE: {average_pe:.2f}')
plt.title(f"{symbol} - 過去五年 PE Ratio 走勢")
plt.xlabel("年份")
plt.ylabel("PE Ratio")
plt.legend()
plt.grid(True)
plt.gca().invert_xaxis()
plt.show()

# ====== 畫出 PS Ratio 走勢 ======
plt.figure(figsize=(12, 6))
plt.plot(dates, ps_ratios, marker='o', label='PS Ratio')
plt.axhline(average_ps, color='green', linestyle='--', label=f'平均 PS: {average_ps:.2f}')
plt.title(f"{symbol} - 過去五年 PS Ratio 走勢")
plt.xlabel("年份")
plt.ylabel("PS Ratio")
plt.legend()
plt.grid(True)
plt.gca().invert_xaxis()
plt.show()

# ====== 一秒判斷 高估 or 低估 ======

# 取最新一筆資料
latest_pe = pe_ratios[0]
latest_ps = ps_ratios[0]

print("\n=== 快速估值判斷 ===")
if latest_pe > average_pe:
    print(f"PE目前是 {latest_pe:.2f}，高於5年平均 {average_pe:.2f} ➔ **偏貴**")
else:
    print(f"PE目前是 {latest_pe:.2f}，低於5年平均 {average_pe:.2f} ➔ **偏便宜**")

if latest_ps > average_ps:
    print(f"PS目前是 {latest_ps:.2f}，高於5年平均 {average_ps:.2f} ➔ **偏貴**")
else:
    print(f"PS目前是 {latest_ps:.2f}，低於5年平均 {average_ps:.2f} ➔ **偏便宜**")