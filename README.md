# JUSTupload
import yfinance as yf

# ========== 估價模型 ==========

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

# ========== 自動抓資料 + 估價主程式 ==========

def auto_valuation(symbol):
    ticker = yf.Ticker(symbol)

    # 抓營收與淨利
    income_statement = ticker.financials
    revenue = income_statement.loc["Total Revenue"].iloc[0] / 1e8  # 億美元
    net_income = income_statement.loc["Net Income"].iloc[0] / 1e8  # 億美元

    # 抓現金流表，自由現金流 (Operating Cash Flow - Capital Expenditures)
    cashflow_statement = ticker.cashflow
    operating_cash_flow = cashflow_statement.loc["Total Cash From Operating Activities"].iloc[0]
    capital_expenditures = cashflow_statement.loc["Capital Expenditures"].iloc[0]
    fcf = (operating_cash_flow + capital_expenditures) / 1e8  # 億美元（注意CapEx是負數）

    # 抓EPS
    eps = ticker.info['trailingEps']

    # 抓流通股數
    shares_outstanding = ticker.info['sharesOutstanding'] / 1e8  # 億股

    # 抓現價
    current_price = ticker.info['currentPrice']

    # 顯示基礎資料
    print(f"\n--- {symbol} 基礎財報資料 ---")
    print(f"營收: {revenue:.2f} 億美元")
    print(f"淨利: {net_income:.2f} 億美元")
    print(f"自由現金流(FCF): {fcf:.2f} 億美元")
    print(f"每股盈餘(EPS): {eps:.2f}")
    print(f"流通股數: {shares_outstanding:.2f} 億股")
    print(f"現價: {current_price} 美元")

    # 設定估價參數
    revenue_growth_rate = 0.05
    ps_ratio = 6
    net_income_growth_rate = 0.05
    pe_ratio = 25
    eps_growth_rate = 0.05
    fcf_growth_rate = 0.08
    discount_rate = 0.10
    terminal_growth_rate = 0.02
    years = 5

    # 開始估價
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

    print(f"\n--- {symbol} 多角度推估股價 ---")
    for method, price in result.items():
        print(f"{method}：{price} 美元")

# ========== 執行 ==========

# 直接輸入想估的股票代號
auto_valuation("AAPL")