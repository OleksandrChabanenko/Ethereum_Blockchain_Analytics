# Comprehensive Ethereum Network Overview
This project provides a comprehensive analysis of financial flows and activity on the Ethereum network, leveraging databases from **[Dune Analytics](https://dune.com/)**. The dashboard includes key metrics such as CEX financial flows, DEX trading volume, ETH price dynamics, network activity, transaction fees (Gas Fees), total ETH burned after EIP-1559, and transaction error analysis. The goal of the project is to offer an analytical tool for exploring Ethereum's financial trends, which can be valuable for traders, analysts, and blockchain researchers.

### 🔗 **Live Dashboard: [View on Dune](https://dune.com/chabanenkooleksandr/comprehensive-ethereum-network-overview)**

## Objectives 
- **Analyze Ethereum Price Trends**: Track historical ETH prices, annual changes, and percentage fluctuations to understand market dynamics.
- **Evaluate DEX Trading Volumes**: Examine yearly trading volumes on decentralized exchanges (DEX) to identify market trends and leaders.
- **Assess CEX Financial Flows**: Analyze incoming and outgoing funds on centralized exchanges (CEX) to understand capital movements.
- **Measure Network Activity**: Track the number of unique transaction senders and receivers to gauge user engagement and network growth.
- **Monitor Gas Fees**: Provide insights into current transaction costs in GWEI and their impact on network usage.
- **Track ETH Burn Metrics**: Quantify the total ETH burned since EIP-1559 and analyze yearly burn dynamics.
- **Investigate Transaction Errors**: Identify common errors in Ethereum transactions and calculate error rates to better understand network behavior.

## Data Sources  
This project uses data from the following tables on the **[Dune Analytics](https://dune.com/data)** platform:  
- **prices.usd**: Historical asset prices in USD.  
- **ethereum.transactions**: Data on Ethereum network transactions.  
- **ethereum.blocks**: Information about Ethereum blocks, including base fees.
- **ethereum.traces**: Detailed data on internal calls (traces) within Ethereum transactions. 
- **dex.trades**: Data on decentralized exchange (DEX) trades.  
- **cex.flows**: Asset flows on centralized exchanges (CEX).

## Usage
Visit the **[Dune Dashboard](https://dune.com/chabanenkooleksandr/comprehensive-ethereum-network-overview)** to interact with visualizations and insights.   
Click **Run** on the dashboard page to update the data and get the most recent information.   
All SQL queries are available in the **[queries.sql](queries.sql)** file.

## Queries and Visualizations

![image](https://github.com/user-attachments/assets/c8fa75d7-c9cd-4fce-862f-e5018e8eb2a8)

### Ethereum Price Tracker
```SQL
SELECT
    price AS eth_price
FROM
    prices.usd
WHERE
    blockchain = 'ethereum'
    AND symbol = 'WETH'
ORDER BY
    minute DESC
LIMIT 1
```

### Ethereum Gas Fees Today
```SQL
SELECT
    DATE(block_time) AS day,
    APPROX_PERCENTILE(gas_price, 0.5) / 1e9 AS median_gas_price_gwei
FROM
    ethereum.transactions
WHERE
    DATE(block_time) = CURRENT_DATE
GROUP BY
    1
```

### Cumulative ETH Burned
```SQL
SELECT
    SUM(b.base_fee_per_gas / 1e18 * t.gas_used) AS burned_eth_all_time
FROM
    ethereum.transactions AS t
JOIN
    ethereum.blocks AS b
    ON t.block_number = b.number
WHERE
    t.block_number >= 12965000
```

### Unique Ethereum Users
```SQL
SELECT
    COUNT(DISTINCT t."from") AS unique_users
FROM
    ethereum.transactions AS t
```


![image](https://github.com/user-attachments/assets/f0e41d7d-7c48-4675-bc62-7c7165043fdf)

### ETH Price Evolution by Year
```SQL
WITH yearly_eth_prices AS (
    SELECT
        EXTRACT(YEAR FROM minute) AS year,
        APPROX_PERCENTILE(price, 0.5) AS eth_price
    FROM
        prices.usd
    WHERE
        symbol = 'ETH'
    GROUP BY
        1
    ORDER BY
        1
)

SELECT
    year,
    ROUND(eth_price, 2) AS price,
    ROUND(eth_price - LAG(eth_price) OVER (ORDER BY year), 2) AS price_difference
FROM
    yearly_eth_prices
```

### Yearly Ethereum Price Changes
```SQL
WITH yearly_eth_prices AS (
    SELECT
        EXTRACT(YEAR FROM minute) AS year,
        APPROX_PERCENTILE(price, 0.5) AS eth_price
    FROM
        prices.usd
    WHERE
        symbol = 'ETH'
    GROUP BY
        1
    ORDER BY
        1
)

SELECT
    year,
    ROUND(eth_price, 2) AS price,
    ROUND(eth_price - LAG(eth_price) OVER (ORDER BY year), 2) AS price_difference,
    CASE
        WHEN LAG(eth_price) OVER (ORDER BY year) IS NULL OR eth_price IS NULL THEN ' '
        ELSE FORMAT('%.2f', (eth_price - LAG(eth_price) OVER (ORDER BY year)) / LAG(eth_price) OVER (ORDER BY year) * 100) || '%'
    END AS price_change_percent
FROM
    yearly_eth_prices
```

![image](https://github.com/user-attachments/assets/86977a08-b730-4590-ae44-c54ebadea6c8)

### Yearly Ethereum Price Min, Max & Median
```SQL
SELECT
    EXTRACT(YEAR FROM minute) AS year,
    MAX(price) AS max_price,
    MIN(price) AS min_price,
    APPROX_PERCENTILE(price, 0.5) AS median_price
FROM
    prices.usd
WHERE
    symbol = 'ETH'
    AND price > 0.01
GROUP BY
    1
ORDER BY
    1
```

![image](https://github.com/user-attachments/assets/95f65d2b-b839-47ca-af76-7af18e0f2504)

### Ethereum Unique User Dynamics by Year
```SQL
SELECT
    EXTRACT(YEAR FROM block_time) AS year,
    COUNT(DISTINCT t."from") AS unique_senders,
    COUNT(DISTINCT t."to") AS unique_receivers
FROM
    ethereum.transactions AS t
GROUP BY
    1
ORDER BY
    1
```

### Yearly ETH Burned Since EIP-1559
```SQL
SELECT
    EXTRACT(YEAR FROM t.block_time) AS year,
    SUM(b.base_fee_per_gas / 1e18 * t.gas_used) AS burned_eth
FROM
    ethereum.transactions AS t
JOIN
    ethereum.blocks AS b
    ON t.block_number = b.number
WHERE
    t.block_number >= 12965000
GROUP BY
    1
ORDER BY
    1
```

![image](https://github.com/user-attachments/assets/8967713d-c509-4d8e-9780-7886d83cc2a6)

### Ethereum DEX Trading Volume Dynamics by Year
```SQL
SELECT
    EXTRACT(YEAR FROM block_date) AS year,
    SUM(amount_usd) AS volume_usd
FROM
    dex.trades
WHERE
    blockchain = 'ethereum'
GROUP BY
    1
ORDER BY
    1
```

### DEX Trading Volume on Ethereum by Project
```SQL
SELECT
    project,
    SUM(amount_usd) AS volume_usd
FROM
    dex.trades
WHERE
    blockchain = 'ethereum'
GROUP BY
    1
ORDER BY
    2 DESC
```

![image](https://github.com/user-attachments/assets/f92e1d0f-598d-4d7c-94bb-5514b7c7bb76)

### Yearly Inflows & Outflows on Ethereum CEX
```SQL
SELECT
    EXTRACT(YEAR FROM block_time) AS year,
    SUM(amount_usd) FILTER (WHERE flow_type = 'Inflow') AS inflow,
    SUM(amount_usd) FILTER (WHERE flow_type = 'Outflow') AS outflow
FROM
    cex.flows
WHERE
    blockchain = 'ethereum'
GROUP BY
    1
```

### Asset Flows on Ethereum CEX: Exchange Comparison
```SQL
SELECT
    cex_name,
    ABS(SUM(amount_usd)) AS abs_usd_flows
FROM
    cex.flows
WHERE
    blockchain = 'ethereum'
GROUP BY
    1
ORDER BY
    2 DESC
```

![image](https://github.com/user-attachments/assets/8c1777e9-1312-4c9e-beb4-915af5ce798c)


### Top 3 Ethereum Transaction Errors in 2025
```SQL
WITH error_counts AS (
    SELECT
        error,
        COUNT(*) AS total_count,
        ROW_NUMBER() OVER (ORDER BY COUNT(*) DESC) AS error_rank
    FROM
        ethereum.traces
    WHERE
        error IS NOT NULL
        AND EXTRACT(YEAR FROM block_date) = 2025
    GROUP BY
        error
)

SELECT
    error,
    total_count AS error_count
FROM
    error_counts
WHERE
    error_rank <= 3

UNION ALL

SELECT
    'Other' AS error,
    SUM(total_count) AS error_count
FROM
    error_counts
WHERE
    error_rank > 3
GROUP BY
    1
```

### Ethereum Network Error in 2025
```SQL
SELECT
    COUNT(*) AS total_count
FROM
    ethereum.traces
WHERE
    error IS NOT NULL
    AND EXTRACT(YEAR FROM block_date) = 2025
```

### Percentage of Failed Ethereum Transactions in 2025
```SQL
WITH error_stats AS (
    SELECT
        COUNT(*) FILTER (WHERE error IS NOT NULL) AS error_count,
        COUNT(*) FILTER (WHERE error IS NULL) AS success_count,
        COUNT(*) AS total_count
    FROM
        ethereum.traces
    WHERE
        EXTRACT(YEAR FROM block_date) = 2025
)

SELECT
    (error_count * 100.0 / total_count) AS error_percentage
FROM
    error_stats
```

### Yearly Percentage of Failed Ethereum Transactions
```SQL
SELECT
    EXTRACT(YEAR FROM block_date) AS year,
    COUNT(*) FILTER (WHERE error IS NOT NULL) * 100.0 / COUNT(*) AS error_percentage
FROM
    ethereum.traces
WHERE
    EXTRACT(YEAR FROM block_date) >= 2017
GROUP BY
    1
ORDER BY
    1
```

## 📝 Insights & Conclusions

### This analysis of the Ethereum network provides several key insights:

**1. Price Dynamics and Market Changes**   
- The most significant ETH price increase occurred in 2021, while the largest drop happened in 2022.   
- The minimum, average, and maximum ETH prices show high volatility, reflecting the cyclical nature of the market.
  
**2. User Activity and Trading Volumes**  
- The number of unique addresses in the network has grown to over 222 million, confirming the scale of the Ethereum ecosystem.   
- Trading volumes on DEX have significantly increased since 2020, indicating a gradual shift of liquidity from CEX.   
- The largest CEX by volume are Binance, Paxos, Bitfinex, and Coinbase.
  
**3. Impact of EIP-1559 on the Network**  
- A total of 4.57 million ETH has been burned through the EIP-1559 mechanism, affecting Ethereum's issuance policy.   
- The volume of ETH burned varies from year to year, reflecting network activity and the level of smart contract usage.
  
**4. Transaction Errors**   
- A total of 2.95 million transaction errors (~0.7% of all operations) were recorded, which is an acceptable rate for a blockchain network of this scale.   
- The most common transaction errors remain insufficient balance, incorrect input data, and exceeding the gas limit.

### Overall Conclusion

This project highlights the dynamic nature of the Ethereum network, showcasing its growth, challenges, and opportunities. Through the analysis of price trends, trading volumes, network activity, gas fees, ETH burn metrics, and transaction errors, we gain a comprehensive understanding of Ethereum's ecosystem. Key takeaways include the increasing adoption of decentralized finance (DeFi), the impact of EIP-1559 on ETH supply, and the importance of monitoring network reliability. These insights are valuable for analysts, investors, and blockchain enthusiasts aiming to navigate the evolving crypto landscape.

## Future Improvements

- Add more data sources (e.g., NFTs, Layer 2).
- Compare Ethereum with other blockchains.
