# Global Housing Market Analysis
### Portfolio Project – Operations / Reporting Data Analyst

## Project Overview
This project analyzes a global house purchase dataset to understand **market affordability, purchase behavior, customer experience, and risk exposure** across countries and cities.
The analysis is structured around four operationally relevant problems that mirror real-world reporting and decision-support scenarios.

The goal is not prediction, but **clear, evidence-based insights** that support market assessment, prioritization, and risk awareness.

## Problem 1: Affordability & Property Size Segmentation
### Business Question
Which markets are more affordable when property size is considered, and how does affordability vary within countries?

```
country_affordability = (
    df_copy.groupby("country")
    .agg(
        avg_price_per_sqft=("price_per_sqft", "mean"),
        avg_price_to_income=("price_to_income_ratio", "mean"),
        avg_salary=("customer_salary", "mean"),
    )
    .reset_index()
)

country_affordability.sort_values(by="avg_price_to_income")
```
![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-1.png)

#### Insight
* Germany, Australia, and Canada rank as the **most affordable markets** based on the lowest average price-to-income ratios.

* India and Singapore appear affordable in raw pricing but become **least affordable once income is considered,** driven by low salary alignment or extreme property pricing.

* Countries with similar average salaries show **meaningful affordability differences,** confirming that income alone does not explain affordability.


## Problem 1 (Extended): Affordability by Property Size Category
### Analytical Question
Within each country, how does affordability change across property sizes?

```
bins = [0, 1000, 2500, 6000]
labels = ["Small", "Medium", "Large"]

df_copy["property_size_category"] = pd.cut(
    df_copy["property_size_sqft"], bins=bins, labels=labels, include_lowest=True
)

size_affordability = (
    df_copy.groupby(["country", "property_size_category"], observed=True)
    .agg(
        avg_price_per_sqft=("price_per_sqft", "mean"),
        avg_price_to_income=("price_to_income_ratio", "mean"),
    )
    .reset_index()
)

size_affordability.sort_values(
    by=["country", "avg_price_to_income"], ascending=[True, True]
)
```

![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-2-1.png)

![Global Housing Market Analysis-Continuation](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-2-2.png)

#### Insights
* In every country, small properties consistently show the lowest price-to-income ratios.

* Price per square foot remains stable across size categories, indicating that affordability strain is driven by size, not pricing inefficiency.

* Larger properties show exponential affordability pressure, especially in India and Singapore.


## Problem 2: Purchase Decision Drivers
### Business Question
What financial factors influence whether a customer proceeds with a property purchase?

```
df_copy["decision"].value_counts(normalize=True)

decision_summary = df_copy.groupby("decision").agg(
    avg_salary=("customer_salary", "mean"),
    avg_price_to_income=("price_to_income_ratio", "mean"),
    avg_emi_ratio=("emi_to_income_ratio", "mean"),
    avg_down_payment=("down_payment", "mean"),
)
```
![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-3.png)

#### Insights
* Buyers earn higher salaries but, more importantly, face **significantly lower affordability pressure.**

* EMI-to-income ratios differ sharply between buyers and non-buyers, while down payment differences are minimal.

* Affordability ratios outperform raw salary in explaining purchase behavior.


### EMI Band Segmentation

```
bins = [0, 0.30, 0.45, 1.0, df["emi_to_income_ratio"].max()]
labels = ["Low", "Medium", "High", "Extreme"]

df_copy["emi_band"] = pd.cut(
    df["emi_to_income_ratio"], bins=bins, labels=labels, include_lowest=True
)

emi_conversion = (
    df_copy.groupby(["country", "emi_band"], observed=True)["decision"]
    .mean()
    .reset_index(name="purchase_rate")
)
```
![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-4-1.png)

![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-4-2.png)

#### Insights
* Purchases occur **only** in Low and Medium EMI bands across all countries.

* Once EMI exceeds ~45% of income, **purchase activity drops to zero.**

* This threshold behavior is consistent globally, indicating structural affordability limits.


## Problem 3: Location Quality & Customer Satisfaction
### Business Question
Which cities deliver the highest customer satisfaction, and what factors contribute to it?

```
city_satisfaction = (
    df_copy.groupby(["country", "city"])
    .agg(
        avg_satisfaction=("satisfaction_score", "mean"),
        avg_neighbourhood=("neighbourhood_rating", "mean"),
        avg_connectivity=("connectivity_score", "mean"),
        property_count=("property_id", "count"),
    )
    .reset_index()
)

top_cities = city_satisfaction.sort_values(
    by="avg_satisfaction", ascending=False
)
```
![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-5-1.png)

![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-5-2.png)

#### Insights
* Top-performing cities include Hyderabad, Delhi, Abu Dhabi, and Houston.

* Traditional global hubs do not consistently dominate satisfaction rankings.

* High satisfaction appears across both emerging and developed markets.


### Contribution Analysis
```
city_satisfaction[
    ["avg_satisfaction", "avg_neighbourhood", "avg_connectivity"]
].corr()
```
![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-6.png)

#### Insights
* Connectivity shows a modest positive relationship with satisfaction.

* Neighbourhood ratings alone do not strongly predict satisfaction.

* Satisfaction is multi-dimensional and not driven by a single factor.

## Problem 4: Market Risk & Price Stability
### Business Question
Which markets exhibit higher housing price volatility and operational risk?

```
country_price_stats = (
    df_copy.groupby("country")
    .agg(
        avg_price=("price", "mean"),
        price_std=("price", "std"),
        min_price=("price", "min"),
        max_price=("price", "max"),
        property_count=("property_id", "count"),
    )
    .reset_index()
)

country_price_stats["price_volatility_ratio"] = (
    country_price_stats["price_std"] / country_price_stats["avg_price"]
)

price_risk_markets = country_price_stats.sort_values(
    by="price_volatility_ratio", ascending=False
)
```
![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-7.png)

#### Insights
* All markets show similar volatility levels once normalized.

* UAE, South Africa, and Japan rank highest in price volatility.

* Normalization is essential for fair cross-market risk comparison.

### Operational Risk Analysis
```
risk_correlation = df_copy[
    ["crime_cases_reported", "legal_cases_on_property", "decision"]
].corr()
```

![Global Housing Market Analysis](Project-1_Global Housing Market Analysis\Images\Global Housing Market Analysis-8.png)

#### Insights
* Legal risk has a stronger negative relationship with purchases than crime.

* Buyers are more sensitive to legal uncertainty than safety perception.

* Risk directly affects demand, not just market stability.

## Conclusion: Cross-Market Insights & Operational Takeaways

This Global Housing Market Analysis demonstrates how affordability, purchasing behavior, customer experience, and market risk must be evaluated together to support informed operational decision-making.

Across countries, affordability analysis confirms that **raw property prices are insufficient indicators** of accessibility. Normalizing prices by income reveals materially different affordability rankings, with several markets appearing attractive on the surface but imposing high affordability pressure on buyers once income is considered. Further segmentation by property size shows that affordability constraints intensify predictably as property size increases, reinforcing the need for size-aware reporting rather than reliance on national averages.

Purchase decision analysis highlights affordability stress — particularly **EMI-to-income ratios** — as the dominant driver of conversion. While buyer income levels are higher on average, income alone does not explain purchasing behavior. Instead, the analysis identifies a consistent affordability threshold beyond which purchasing activity collapses across all markets, providing a clear operational signal for sales, lending, and policy reporting.

Location-based analysis demonstrates that **customer satisfaction varies meaningfully at the city level,** even within the same country. High satisfaction is not exclusive to global metropolitan hubs and is only partially explained by neighborhood quality. Connectivity shows a stronger association with satisfaction, suggesting infrastructure and accessibility play a more influential role in shaping customer experience than reputation alone.

Market risk assessment reveals that price volatility is broadly consistent across countries once normalized, emphasizing the importance of relative rather than absolute risk metrics. However, operational risk factors — particularly legal exposure — show a measurable negative relationship with purchasing decisions. The combined risk analysis identifies markets that sustain demand despite elevated price volatility and operational risk, representing **attractive but fragile environments** that require active monitoring and mitigation rather than uninformed expansion.

Taken together, this analysis demonstrates an end-to-end analytical workflow that mirrors real-world operations and reporting contexts:

* Framing business-relevant questions

* Designing interpretable metrics

* Producing diagnostic (not predictive) insights

* Translating analysis into operational implications

This project reflects how data analysis supports **market assessment, prioritization, and risk-aware decision-making,** rather than serving as a purely descriptive or academic exercise.