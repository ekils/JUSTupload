# JUSTupload
import requests

# Rapid API key 設定
API_KEY = "你的RapidAPI_KEY"

# 股票代號
symbol = "AAPL"

# Rapid API headers
headers = {
    "X-RapidAPI-Key": API_KEY,
    "X-RapidAPI-Host": "financial-modeling-prep.p.rapidapi.com"
}

# 收入與淨利
income_url = f"https://financial-modeling-prep.p.rapidapi.com/v3/income-statement/{symbol}?limit=1&apikey={API_KEY}"
income_data = requests.get(income_url, headers=headers).json()

# 自由現金流
cashflow_url = f"https://financial-modeling-prep.p.rapidapi.com/v3/cash-flow-statement/{symbol}?limit=1&apikey={API_KEY}"
cashflow_data = requests.get(cashflow_url, headers=headers).json()

# 流通股數
metrics_url = f"https://financial-modeling-prep.p.rapidapi.com/v3/key-metrics/{symbol}?limit=1&apikey={API_KEY}"
metrics_data = requests.get(metrics_url, headers=headers).json()

# 整理資料
revenue = income_data[0]['revenue'] / 1e8  # 億美元
net_income = income_data[0]['netIncome'] / 1e8  # 億美元
eps = income_data[0]['eps']
shares_outstanding = metrics_data[0]['sharesOutstanding'] / 1e8  # 億股
fcf = cashflow_data[0]['freeCashFlow'] / 1e8  # 億美元

print(f"營收: {revenue:.2f} 億美元")
print(f"淨利: {net_income:.2f} 億美元")
print(f"每股盈餘(EPS): {eps}")
print(f"自由現金流(FCF): {fcf:.2f} 億美元")
print(f"流通股數: {shares_outstanding:.2f} 億股")