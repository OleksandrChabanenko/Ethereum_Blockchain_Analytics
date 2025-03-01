# Ethereum-Blockchain-Analytics
Comprehensive dashboard for monitoring Ethereum blockchain metrics, financial flows, and network activity in real-time.

![image](https://github.com/user-attachments/assets/c8fa75d7-c9cd-4fce-862f-e5018e8eb2a8)

```SQL
--Ethereum Price Tracker
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

--Ethereum Gas Fees Today
SELECT
    DATE(block_time) AS day,
    APPROX_PERCENTILE(gas_price, 0.5) / 1e9 AS median_gas_price_gwei
FROM
    ethereum.transactions
WHERE
    DATE(block_time) = CURRENT_DATE
GROUP BY
    1

--Cumulative ETH Burned
SELECT
    SUM(b.base_fee_per_gas / 1e18 * t.gas_used) AS burned_eth_all_time
FROM
    ethereum.transactions AS t
JOIN
    ethereum.blocks AS b
    ON t.block_number = b.number
WHERE
    t.block_number >= 12965000

--Unique Ethereum Users
SELECT
    COUNT(DISTINCT t."from") AS unique_users
FROM
    ethereum.transactions AS t
```

![image](https://github.com/user-attachments/assets/f0e41d7d-7c48-4675-bc62-7c7165043fdf)

```SQL
--ETH Price Evolution by Year
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

--Yearly Ethereum Price Changes
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
```SQL
--Yearly Ethereum Price Min, Max & Median
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
```SQL
--Ethereum Unique User Dynamics by Year
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

--Yearly ETH Burned Since EIP-1559
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

```SQL
--Ethereum DEX Trading Volume Dynamics by Year
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

--DEX Trading Volume on Ethereum by Project
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
```SQL
--SELECT
    EXTRACT(YEAR FROM block_time) AS year,
    SUM(amount_usd) FILTER (WHERE flow_type = 'Inflow') AS inflow,
    SUM(amount_usd) FILTER (WHERE flow_type = 'Outflow') AS outflow
FROM
    cex.flows
WHERE
    blockchain = 'ethereum'
GROUP BY
    1

--Asset Flows on Ethereum CEX: Exchange Comparison
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
```SQL
--Top 3 Ethereum Transaction Errors in 2025
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

--Ethereum Network Error in 2025
SELECT
    COUNT(*) AS total_count
FROM
    ethereum.traces
WHERE
    error IS NOT NULL
    AND EXTRACT(YEAR FROM block_date) = 2025

--Percentage of Failed Ethereum Transactions in 2025
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

--Yearly Percentage of Failed Ethereum Transactions
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
