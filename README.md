from bs4 import BeautifulSoup
import requests

url = "https://asaxiy.uz/product/televizory-i-aksessuary/televizory"

headers = {
    "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/141.0.0.0 Safari/537.36"
}

sorov = requests.get(url, headers=headers)
data = sorov.text

product_list = []

soup = BeautifulSoup(data, "html.parser")

main_block = soup.find("div", class_="row")
block = main_block.find_all("div", class_="col-6")

for product in block:
    try:
        image = product.find("img", class_="img-fluid")["data-src"]
        product_name = product.find("span", class_="product__item__info-title").get_text(strip=True)
        product_price = product.find("span", class_="product__item-price").get_text(strip=True)
        product_credit = product.find("div", class_="installment__price").get_text(strip=True)

        product_list.append({
            "Televizor nomi": product_name,
            "Rasm": image,
            "Narxi": product_price,
            "Kredit": product_credit
        })
    except Exception:
        continue

for p in product_list:
    print(p)
