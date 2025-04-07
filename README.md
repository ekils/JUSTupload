def valuation_tool(
    revenue, ps_ratio,
    net_income, pe_ratio,
    eps,
    shares_outstanding,
    fcf, fcf_growth_rate, discount_rate, terminal_growth_rate,
    years=5
):
    # PS Model (直接用現在營收)
    ps_valuation = (revenue * ps_ratio) / shares_outstanding

    # PE Model (直接用現在淨利)
    pe_valuation = (net_income * pe_ratio) / shares_outstanding

    # EPS Model (直接用現在EPS)
    eps_valuation = eps * pe_ratio

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
    dcf_valuation = enterprise_value / shares_outstanding

    return {
        "PS模型股價推估": round(ps_valuation, 2),
        "PE模型股價推估": round(pe_valuation, 2),
        "EPS模型股價推估": round(eps_valuation, 2),
        "DCF模型股價推估": round(dcf_valuation, 2),
    }



# 範例自動抓資料 + 丟估價
result = valuation_tool(
    revenue=revenue,
    ps_ratio=6,
    net_income=net_income,
    pe_ratio=25,
    eps=eps,
    shares_outstanding=shares_outstanding,
    fcf=fcf,
    fcf_growth_rate=0.08,
    discount_rate=0.10,
    terminal_growth_rate=0.02,
    years=5
)

