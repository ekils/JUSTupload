# JUSTupload
import requests

# ====== 設定區 ======

API_KEY = '你的RapidAPI_KEY'  # <<<< 記得換成你自己的Rapid API Key
symbol = 'AAPL'              # 想估價的股票代碼
years = 5                    # 估價年數

# 預設參數（可以自己調）
revenue_growth_rate = 0.05
ps_ratio = 6
net_income_growth_rate = 0.05
pe_ratio = 25
eps_growth_rate = 0.05
fcf_growth_rate = 0.08
discount_rate = 0.10
terminal_growth_rate = 0.02

# ====== API 抓資料區 ======

headers = {
    "X-RapidAPI-Key": API_KEY,
    "X-RapidAPI-Host": "financialmodelingprep.p.rapidapi.com"
}

# 抓自由現金流
cashflow_url = f"https://financialmodelingprep.p.rapidapi.com/cash-flow-statement/{symbol}?limit=1"
cashflow_data = requests.get(cashflow_url, headers=headers).json()

# 抓營收與淨利
income_url = f"https://financialmodelingprep.p.rapidapi.com/income-statement/{symbol}?limit=1"
income_data = requests.get(income_url, headers=headers).json()

# 抓股數與即時股價
profile_url = f"https://financialmodelingprep.p.rapidapi.com/profile/{symbol}"
profile_data = requests.get(profile_url, headers=headers).json()

# 整理數據
revenue = income_data[0]['revenue'] / 1e8  # 億美元
net_income = income_data[0]['netIncome'] / 1e8  # 億美元
eps = income_data[0]['eps']
shares_outstanding = profile_data[0]['sharesOutstanding'] / 1e8  # 億股
fcf = cashflow_data[0]['freeCashFlow'] / 1e8  # 億美元
current_price = profile_data[0]['price']

print(f"--- {symbol} 財務摘要 ---")
print(f"營收: {revenue:.2f} 億美元")
print(f"淨利: {net_income:.2f} 億美元")
print(f"每股盈餘(EPS): {eps}")
print(f"自由現金流(FCF): {fcf:.2f} 億美元")
print(f"流通股數: {shares_outstanding:.2f} 億股")
print(f"現價: {current_price} 美元")
print()

# ====== 估值模型區 ======

def valuation_tool(
    revenue, revenue_growth_rate, ps_ratio,
    net_income, net_income_growth_rate, pe_ratio,
    eps, eps_growth_rate,
    shares_outstanding,
    fcf, fcf_growth_rate, discount_rate, terminal_growth_rate,
    years=5
):
    # PS Model (營收推估)
    future_revenue = revenue * (1 + revenue_growth_rate) ** years
    ps_valuation = (future_revenue * ps_ratio) / (shares_outstanding * 1e8)

    # PE Model (淨利推估)
    future_net_income = net_income * (1 + net_income_growth_rate) ** years
    pe_valuation = (future_net_income * pe_ratio) / (shares_outstanding * 1e8)

    # EPS Model (EPS推估)
    future_eps = eps * (1 + eps_growth_rate) ** years
    eps_valuation = future_eps * pe_ratio

    # DCF Model (自由現金流折現推估)
    discounted_cash_flows = 0
    current_fcf = fcf

    for year in range(1, years + 1):
        current_fcf *= (1 + fcf_growth_rate)
        discounted_fcf = current_fcf / ((1 + discount_rate) ** year)
        discounted_cash_flows += discounted_fcf

    # Terminal Value
    terminal_value = current_fcf * (1 + terminal_growth_rate) / (discount_rate - terminal_growth_rate)
    discounted_terminal_value = terminal_value / ((1 + discount_rate) ** years)

    # Enterprise Value
    enterprise_value = discounted_cash_flows + discounted_terminal_value
    dcf_valuation = enterprise_value / (shares_outstanding * 1e8)

    return {
        "PS模型股價推估": round(ps_valuation, 2),
        "PE模型股價推估": round(pe_valuation, 2),
        "EPS模型股價推估": round(eps_valuation, 2),
        "DCF模型股價推估": round(dcf_valuation, 2),
    }

# 執行估值
result = valuation_tool(
    revenue=revenue,
    revenue_growth_rate=revenue_growth_rate,
    ps_ratio=ps_ratio,
    net_income=net_income,
    net_income_growth_rate=net_income_growth_rate,
    pe_ratio=pe_ratio,
    eps=eps,
    eps_growth_rate=eps_growth_rate,
    shares_outstanding=shares_outstanding,
    fcf=fcf,
    fcf_growth_rate=fcf_growth_rate,
    discount_rate=discount_rate,
    terminal_growth_rate=terminal_growth_rate,
    years=years
)

print(f"--- {symbol} 多角度推估股價 ---")
for method, price in result.items():
    print(f"{method}：{price} 美元")