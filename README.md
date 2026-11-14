
# ============================================================

st.set_page_config(
    page_title="Sales Analytics Dashboard",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded"
)


# ============================================================
# DARK THEME
# ============================================================

st.markdown("""
<style>

.stApp {
    background-color: #07111f;
    color: white;
}

[data-testid="stSidebar"] {
    background-color: #0d1726;
}

[data-testid="stSidebar"] * {
    color: white;
}

h1, h2, h3 {
    color: white !important;
}

p {
    color: #c7d1df;
}

/* Metric cards */

[data-testid="stMetric"] {
    background-color: #111c2d;
    border: 1px solid #33465f;
    border-radius: 14px;
    padding: 18px;
}

[data-testid="stMetricLabel"] {
    color: #c7d1df !important;
}

[data-testid="stMetricValue"] {
    color: white !important;
}

/* Tabs */

button[data-baseweb="tab"] {
    color: white !important;
}

/* Dataframe */

[data-testid="stDataFrame"] {
    border: 1px solid #33465f;
}

/* Divider */

hr {
    border-color: #2b3c52;
}

</style>
""", unsafe_allow_html=True)


# ============================================================
# LOAD DATA
# ============================================================

@st.cache_data
def load_data():

    data = pd.read_csv("sales_data.csv", sep="\t")

    data.columns = data.columns.str.strip()

    data["Sale_Date"] = pd.to_datetime(
        data["Sale_Date"],
        errors="coerce"
    )

    data = data.dropna(subset=["Sale_Date"])

    return data


df = load_data()


# ============================================================
# TITLE
# ============================================================

st.title("📊 Sales Analytics Dashboard")

st.write(
    "Explore sales performance, products, regions, customers "
    "and sales channels."
)

st.divider()


# ============================================================
# SIDEBAR FILTERS
# ============================================================

st.sidebar.title("🔎 Filters")

st.sidebar.markdown("### 📅 Date Range")

min_date = df["Sale_Date"].min().date()
max_date = df["Sale_Date"].max().date()

date_range = st.sidebar.date_input(
    "Select date range",
    value=(min_date, max_date),
    min_value=min_date,
    max_value=max_date
)


st.sidebar.markdown("### 🌍 Region")

regions = sorted(
    df["Region"].dropna().unique()
)

selected_regions = st.sidebar.multiselect(
    "Select Region",
    regions,
    default=regions
)


st.sidebar.markdown("### 👤 Sales Representative")

sales_reps = sorted(
    df["Sales_Rep"].dropna().unique()
)

selected_reps = st.sidebar.multiselect(
    "Select Sales Rep",
    sales_reps,
    default=sales_reps
)


st.sidebar.markdown("### 📦 Product Category")

categories = sorted(
    df["Product_Category"].dropna().unique()
)

selected_categories = st.sidebar.multiselect(
    "Select Category",
    categories,
    default=categories
)


st.sidebar.markdown("### 🛒 Sales Channel")

channels = sorted(
    df["Sales_Channel"].dropna().unique()
)

selected_channels = st.sidebar.multiselect(
    "Select Channel",
    channels,
    default=channels
)


st.sidebar.markdown("### 👥 Customer Type")

customers = sorted(
    df["Customer_Type"].dropna().unique()
)

selected_customers = st.sidebar.multiselect(
    "Customer Type",
    customers,
    default=customers
)


# ============================================================
# APPLY FILTERS
# ============================================================

filtered_df = df.copy()


# Date filter

if len(date_range) == 2:

    start_date = pd.to_datetime(date_range[0])

    end_date = pd.to_datetime(date_range[1])

    filtered_df = filtered_df[
        (filtered_df["Sale_Date"] >= start_date)
        &
        (filtered_df["Sale_Date"] <= end_date)
    ]


# Region

filtered_df = filtered_df[
    filtered_df["Region"].isin(selected_regions)
]


# Sales Rep

filtered_df = filtered_df[
    filtered_df["Sales_Rep"].isin(selected_reps)
]


# Category

filtered_df = filtered_df[
    filtered_df["Product_Category"].isin(
        selected_categories
    )
]


# Channel

filtered_df = filtered_df[
    filtered_df["Sales_Channel"].isin(
        selected_channels
    )
]


# Customer

filtered_df = filtered_df[
    filtered_df["Customer_Type"].isin(
        selected_customers
    )
]


# ============================================================
# KPI CALCULATIONS
# ============================================================

total_sales = filtered_df["Sales_Amount"].sum()

total_quantity = filtered_df["Quantity_Sold"].sum()

average_sale = filtered_df["Sales_Amount"].mean()

total_orders = len(filtered_df)


# ============================================================
# KPI SECTION
# ============================================================

st.header("📈 Key Performance Indicators")

col1, col2, col3, col4 = st.columns(4)


with col1:

    st.metric(
        "💰 Total Sales",
        f"${total_sales:,.0f}"
    )


with col2:

    st.metric(
        "📦 Quantity Sold",
        f"{total_quantity:,}"
    )


with col3:

    st.metric(
        "💵 Average Sale",
        f"${average_sale:,.0f}"
    )


with col4:

    st.metric(
        "🧾 Total Transactions",
        f"{total_orders:,}"
    )


st.divider()


# ============================================================
# SALES ANALYSIS
# ============================================================

st.header("💰 Sales Analysis")


col1, col2 = st.columns(2)


# ------------------------------------------------------------
# SALES BY REGION
# ------------------------------------------------------------

with col1:

    region_sales = (
        filtered_df
        .groupby("Region")["Sales_Amount"]
        .sum()
        .reset_index()
        .sort_values(
            "Sales_Amount",
            ascending=False
        )
    )

    fig_region = px.bar(
        region_sales,
        x="Region",
        y="Sales_Amount",
        color="Region",
        text_auto=".2s",
        title="Total Sales by Region"
    )

    fig_region.update_layout(
        template="plotly_dark",
        height=420,
        paper_bgcolor="#101b2b",
        plot_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_region,
        use_container_width=True
    )


# ------------------------------------------------------------
# SALES BY CATEGORY
# ------------------------------------------------------------

with col2:

    category_sales = (
        filtered_df
        .groupby("Product_Category")["Sales_Amount"]
        .sum()
        .reset_index()
    )

    fig_category = px.pie(
        category_sales,
        names="Product_Category",
        values="Sales_Amount",
        hole=0.45,
        title="Sales by Product Category"
    )

    fig_category.update_layout(
        template="plotly_dark",
        height=420,
        paper_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_category,
        use_container_width=True
    )


# ============================================================
# SALES TREND
# ============================================================

st.header("📅 Sales Trend")


daily_sales = (
    filtered_df
    .groupby("Sale_Date")["Sales_Amount"]
    .sum()
    .reset_index()
)


fig_trend = px.line(
    daily_sales,
    x="Sale_Date",
    y="Sales_Amount",
    markers=True,
    title="Daily Sales Trend"
)

fig_trend.update_traces(
    line_color="#00c8ff",
    marker_color="#00c8ff"
)

fig_trend.update_layout(
    template="plotly_dark",
    height=450,
    paper_bgcolor="#101b2b",
    plot_bgcolor="#101b2b"
)

st.plotly_chart(
    fig_trend,
    use_container_width=True
)


# ============================================================
# SALES REP ANALYSIS
# ============================================================

st.header("👤 Sales Representative Performance")


col1, col2 = st.columns(2)


# ------------------------------------------------------------
# SALES REP SALES
# ------------------------------------------------------------

with col1:

    rep_sales = (
        filtered_df
        .groupby("Sales_Rep")["Sales_Amount"]
        .sum()
        .reset_index()
        .sort_values(
            "Sales_Amount",
            ascending=False
        )
    )

    fig_rep = px.bar(
        rep_sales,
        x="Sales_Rep",
        y="Sales_Amount",
        color="Sales_Rep",
        text_auto=".2s",
        title="Sales by Sales Representative"
    )

    fig_rep.update_layout(
        template="plotly_dark",
        height=420,
        paper_bgcolor="#101b2b",
        plot_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_rep,
        use_container_width=True
    )


# ------------------------------------------------------------
# QUANTITY BY SALES REP
# ------------------------------------------------------------

with col2:

    rep_quantity = (
        filtered_df
        .groupby("Sales_Rep")["Quantity_Sold"]
        .sum()
        .reset_index()
    )

    fig_quantity = px.bar(
        rep_quantity,
        x="Sales_Rep",
        y="Quantity_Sold",
        color="Sales_Rep",
        text="Quantity_Sold",
        title="Quantity Sold by Sales Representative"
    )

    fig_quantity.update_layout(
        template="plotly_dark",
        height=420,
        paper_bgcolor="#101b2b",
        plot_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_quantity,
        use_container_width=True
    )


# ============================================================
# PRODUCT ANALYSIS
# ============================================================

st.header("📦 Product Analysis")


col1, col2 = st.columns(2)


# ------------------------------------------------------------
# CATEGORY QUANTITY
# ------------------------------------------------------------

with col1:

    category_quantity = (
        filtered_df
        .groupby("Product_Category")["Quantity_Sold"]
        .sum()
        .reset_index()
    )

    fig_product = px.bar(
        category_quantity,
        x="Product_Category",
        y="Quantity_Sold",
        color="Product_Category",
        text="Quantity_Sold",
        title="Quantity Sold by Product Category"
    )

    fig_product.update_layout(
        template="plotly_dark",
        height=420,
        paper_bgcolor="#101b2b",
        plot_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_product,
        use_container_width=True
    )


# ------------------------------------------------------------
# PRICE VS QUANTITY
# ------------------------------------------------------------

with col2:

    fig_scatter = px.scatter(
        filtered_df,
        x="Quantity_Sold",
        y="Sales_Amount",
        color="Product_Category",
        size="Discount",
        hover_data=[
            "Sales_Rep",
            "Region",
            "Customer_Type",
            "Sales_Channel"
        ],
        title="Quantity Sold vs Sales Amount"
    )

    fig_scatter.update_layout(
        template="plotly_dark",
        height=420,
        paper_bgcolor="#101b2b",
        plot_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_scatter,
        use_container_width=True
    )


# ============================================================
# CUSTOMER ANALYSIS
# ============================================================

st.header("👥 Customer Analysis")


col1, col2 = st.columns(2)


# ------------------------------------------------------------
# CUSTOMER TYPE
# ------------------------------------------------------------

with col1:

    customer_sales = (
        filtered_df
        .groupby("Customer_Type")["Sales_Amount"]
        .sum()
        .reset_index()
    )

    fig_customer = px.pie(
        customer_sales,
        names="Customer_Type",
        values="Sales_Amount",
        hole=0.45,
        title="Sales by Customer Type"
    )

    fig_customer.update_layout(
        template="plotly_dark",
        height=400,
        paper_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_customer,
        use_container_width=True
    )


# ------------------------------------------------------------
# SALES CHANNEL
# ------------------------------------------------------------

with col2:

    channel_sales = (
        filtered_df
        .groupby("Sales_Channel")["Sales_Amount"]
        .sum()
        .reset_index()
    )

    fig_channel = px.bar(
        channel_sales,
        x="Sales_Channel",
        y="Sales_Amount",
        color="Sales_Channel",
        text_auto=".2s",
        title="Sales by Sales Channel"
    )

    fig_channel.update_layout(
        template="plotly_dark",
        height=400,
        paper_bgcolor="#101b2b",
        plot_bgcolor="#101b2b"
    )

    st.plotly_chart(
        fig_channel,
        use_container_width=True
    )


# ============================================================
# PAYMENT METHOD
# ============================================================

st.header("💳 Payment Analysis")


payment_sales = (
    filtered_df
    .groupby("Payment_Method")["Sales_Amount"]
    .sum()
    .reset_index()
)


fig_payment = px.bar(
    payment_sales,
    x="Payment_Method",
    y="Sales_Amount",
    color="Payment_Method",
    text_auto=".2s",
    title="Sales by Payment Method"
)

fig_payment.update_layout(
    template="plotly_dark",
    height=420,
    paper_bgcolor="#101b2b",
    plot_bgcolor="#101b2b"
)

st.plotly_chart(
    fig_payment,
    use_container_width=True
)


# ============================================================
# DISCOUNT ANALYSIS
# ============================================================

st.header("🏷️ Discount Analysis")


discount_data = (
    filtered_df
    .groupby("Discount")["Sales_Amount"]
    .mean()
    .reset_index()
)


fig_discount = px.scatter(
    discount_data,
    x="Discount",
    y="Sales_Amount",
    size="Sales_Amount",
    title="Discount vs Average Sales Amount"
)

fig_discount.update_traces(
    marker_color="#ff4b4b"
)

fig_discount.update_layout(
    template="plotly_dark",
    height=420,
    paper_bgcolor="#101b2b",
    plot_bgcolor="#101b2b"
)

st.plotly_chart(
    fig_discount,
    use_container_width=True
)


# ============================================================
# TOP SALES
# ============================================================

st.header("🏆 Top Sales Transactions")


top_sales = (
    filtered_df
    .sort_values(
        "Sales_Amount",
        ascending=False
    )
    .head(10)
)


st.dataframe(
    top_sales[
        [
            "Product_ID",
            "Sale_Date",
            "Sales_Rep",
            "Region",
            "Sales_Amount",
            "Quantity_Sold",
            "Product_Category",
            "Customer_Type",
            "Sales_Channel"
        ]
    ],
    use_container_width=True,
    hide_index=True
)


# ============================================================
# RAW DATA
# ============================================================

with st.expander("📋 View Complete Filtered Dataset"):

    st.dataframe(
        filtered_df,
        use_container_width=True,
        hide_index=True
    )


# ============================================================
# FOOTER
# ============================================================

st.divider()

st.caption(
    "📊 Sales Analytics Dashboard | "
    "Built with Python, Pandas, Plotly and Streamlit"
)import streamlit as st
import pandas as pd
import plotly.express as px


# ============================================================
# PAGE CONFIGURATION
