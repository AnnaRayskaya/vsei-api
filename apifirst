from flask import Flask, request, jsonify
from bs4 import BeautifulSoup
import requests
import re

app = Flask(__name__)

BASE_URL = "https://www.vseinstrumenti.ru/search/?q="
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/91.0.4472.124 Safari/537.36"
}


def clean_price(price_str):
    price_digits = re.sub(r"\D", "", price_str)
    return int(price_digits) if price_digits else 0


@app.route("/find", methods=["POST"])
def find_products():
    data = request.json
    query = data.get("тип", "") + " " + " ".join([f"{k} {v}" for k, v in data.get("параметры", {}).items()])

    response = requests.get(BASE_URL + query, headers=HEADERS)
    soup = BeautifulSoup(response.text, "html.parser")
    items = soup.select("div[itemtype='http://schema.org/Product']")

    results = []
    for item in items:
        title_tag = item.select_one(".product-title__text")
        price_tag = item.select_one("meta[itemprop='price']")
        link_tag = item.select_one("a.product-title__text")

        if not (title_tag and price_tag and link_tag):
            continue

        price = clean_price(price_tag.get("content", "0"))
        discounted_price = round(price * 0.95)

        results.append({
            "название": title_tag.text.strip(),
            "ссылка": "https://www.vseinstrumenti.ru" + link_tag.get("href"),
            "цена_на_сайте": price,
            "цена_со_скидкой": discounted_price
        })

    return jsonify(results)


if __name__ == "__main__":
    app.run(debug=True, port=5000)
