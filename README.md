import requests
import json

def get_discount_rate(symbol):
    API_KEY = "你的RapidAPI_KEY"
    url = f"https://financialmodelingprep.p.rapidapi.com/v4/company-valuation/{symbol}?apikey={API_KEY}"

    headers = {
        "X-RapidAPI-Key": API_KEY,
        "X-RapidAPI-Host": "financial-modeling-prep.p.rapidapi.com"
    }

    try:
        response = requests.get(url, headers=headers)

        # 加這個是安全保險
        response.encoding = 'utf-8'

        # 嘗試只取JSON，不print原始文字
        data = response.json()

        if isinstance(data, list) and len(data) > 0:
            wacc = data[0].get('wacc', None)

            if wacc is not None:
                print(f"{symbol} 折現率 (WACC)：{wacc:.2f}%")
                return wacc / 100
            else:
                print(f"⚠️ {symbol} 查無 WACC 資料")
                return None
        else:
            print(f"⚠️ 無法取得 {symbol} 折現率資料")
            return None

    except Exception as e:
        print(f"⚠️ 錯誤：{str(e)}")
        return None