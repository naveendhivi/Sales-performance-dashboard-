import streamlit as st
import pandas as pd
import plotly.express as px

# Page configuration
st.set_page_config(
    page_title="Sales Performance Dashboard",
    page_icon="📊",
    layout="wide"
)

st.title("📊 Sales Performance Dashboard")

# Load dataset
df = pd.read_csv("sales_data.csv")

# Convert date column
df["Date"] = pd.to_datetime(df["Date"])

# KPIs
total_sales = df["Sales"].sum()
total_profit = df["Profit"].sum()
total_orders = df["Order_ID"].nunique()
profit_margin = (total_profit / total_sales) * 100

col1, col2, col3, col4 = st.columns(4)

col1.metric("Total Sales", f"₹{total_sales:,.0f}")
col2.metric("Total Profit", f"₹{total_profit:,.0f}")
col3.metric("Total Orders", total_orders)
col4.metric("Profit Margin", f"{profit_margin:.2f}%")

st.divider()

# Filters
region = st.sidebar.multiselect(
    "Select Region",
    df["Region"].unique(),
    default=df["Region"].unique()
)

category = st.sidebar.multiselect(
    "Select Category",
    df["Category"].unique(),
    default=df["Category"].unique()
)

filtered_df = df[
    (df["Region"].isin(region)) &
    (df["Category"].isin(category))
]

# Sales trend
monthly_sales = (
    filtered_df
    .groupby(filtered_df["Date"].dt.to_period("M"))["Sales"]
    .sum()
    .reset_index()
)

monthly_sales["Date"] = monthly_sales["Date"].astype(str)

fig1 = px.line(
    monthly_sales,
    x="Date",
    y="Sales",
    title="Monthly Sales Trend",
    markers=True
)

st.plotly_chart(fig1, use_container_width=True)

# Regional sales
regional_sales = (
    filtered_df.groupby("Region")["Sales"]
    .sum()
    .reset_index()
)

fig2 = px.bar(
    regional_sales,
    x="Region",
    y="Sales",
    title="Sales by Region"
)

st.plotly_chart(fig2, use_container_width=True)

# Category sales
category_sales = (
    filtered_df.groupby("Category")["Sales"]
    .sum()
    .reset_index()
)

fig3 = px.pie(
    category_sales,
    names="Category",
    values="Sales",
    title="Sales by Category"
)

st.plotly_chart(fig3, use_container_width=True)

# Top products
top_products = (
    filtered_df.groupby("Product")["Sales"]
    .sum()
    .sort_values(ascending=False)
    .head(10)
    .reset_index()
)

st.subheader("🏆 Top 10 Products")

st.dataframe(
    top_products,
    use_container_width=True,
    hide_index=True
)
