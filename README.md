MinerviniBot - Streamlit App for Indian Stocks

Requirements: streamlit, yfinance, fpdf, pandas

import yfinance as yf import streamlit as st from fpdf import FPDF import pandas as pd

st.set_page_config(page_title="MinerviniBot", layout="centered") st.title("MinerviniBot: Indian Stock Analyzer")

Input stock symbol

stock_symbol = st.text_input("Enter NSE Stock Symbol (e.g., CDSL, TCS, INFY)", value="CDSL")

Checklist function

def minervini_checklist(symbol): try: stock = yf.Ticker(symbol + ".NS") info = stock.info

checklist = {
        "ROE > 15%": info.get("returnOnEquity", 0) and info["returnOnEquity"] * 100 > 15,
        "Positive EPS": info.get("trailingEps", 0) > 0,
        "P/E < 40": info.get("trailingPE", 0) < 40,
        "Profit Margin > 10%": info.get("profitMargins", 0) and info["profitMargins"] * 100 > 10,
        "Revenue Growth > 10%": info.get("revenueGrowth", 0) and info["revenueGrowth"] * 100 > 10,
    }

    score = sum(checklist.values())
    total = len(checklist)
    decision = "YES" if score == total else ("PARTIALLY YES" if score >= total * 0.6 else "NO")
    return checklist, decision, info

except Exception as e:
    return {}, "ERROR", {"error": str(e)}

PDF Report function

def generate_pdf(symbol, checklist, decision): pdf = FPDF() pdf.add_page() pdf.set_font("Arial", size=12) pdf.cell(200, 10, txt=f"Minervini Report: {symbol}", ln=True) pdf.cell(200, 10, txt=f"Decision: {decision}", ln=True) pdf.ln(10) for k, v in checklist.items(): pdf.cell(200, 10, txt=f"{k}: {'Yes' if v else 'No'}", ln=True) filename = f"{symbol}_report.pdf" pdf.output(filename) return filename

if stock_symbol: with st.spinner("Analyzing..."): checklist, decision, info = minervini_checklist(stock_symbol)

if decision != "ERROR":
    st.subheader(f"Decision: {decision}")
    for k, v in checklist.items():
        st.write(f"- {k}: {'Yes' if v else 'No'}")

    if st.button("Download PDF Report"):
        pdf_file = generate_pdf(stock_symbol, checklist, decision)
        with open(pdf_file, "rb") as f:
            st.download_button("Download", f, file_name=pdf_file)
else:
    st.error("Failed to fetch stock data. Please check symbol or try again later.")

