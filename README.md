import requests

def get_discount_rate(symbol):
    """
    輸入股票代碼，回傳該公司的折現率 (WACC)。
    需要使用 Rapid API Financial Modeling Prep (FMP) 的 v4 company-valuation 端點。
    """
    API_KEY = "你的RapidAPI_KEY"  # <<< 這邊換成你的Key
    url = f"https://financialmodelingprep.p.rapidapi.com/v4/company-valuation/{symbol}?apikey={API_KEY}"

    headers = {
        "X-RapidAPI-Key": API_KEY,
        "X-RapidAPI-Host": "financial-modeling-prep.p.rapidapi.com"
    }

    try:
        response = requests.get(url, headers=headers)
        data = response.json()

        # 這個 API 回傳的是一個列表
        if isinstance(data, list) and len(data) > 0:
            wacc = data[0].get('wacc', None)

            if wacc is not None:
                print(f"{symbol} 折現率 (WACC)：{wacc:.2f}%")
                return wacc / 100  # 回傳小數格式，例如 0.08 代表8%
            else:
                print(f"⚠️ {symbol} 查無 WACC 資料，請手動設定！")
                return None
        else:
            print(f"⚠️ 無法取得 {symbol} 折現率資料")
            return None

    except Exception as e:
        print(f"⚠️ 錯誤：{e}")
        return None