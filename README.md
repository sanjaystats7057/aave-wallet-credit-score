# aave-wallet-credit-score
This project uses transaction-level data from the Aave V2 protocol to assign a credit score (0–1000) to wallets based on their behavior. The model analyzes patterns like deposits, borrows, repayments, liquidations, and redemptions to classify wallets as reliable or risky. 
# Aave Wallet Credit Scoring

This project assigns a credit score (0 to 1000) to each wallet based on its historical activity in the Aave V2 protocol. It uses raw JSON transaction data and a scoring logic based on responsible vs. risky behavior.

---

##  Problem Statement

You are provided with sample raw, transaction-level data from the Aave V2 protocol. Each record corresponds to a wallet interacting through actions such as:
- `deposit`
- `borrow`
- `repay`
- `redeemunderlying`
- `liquidationcall`

The goal is to develop a model that scores each wallet between 0 (risky/irresponsible) and 1000 (safe/responsible) based on their transaction behavior.

---
Data Cleaning
The raw transaction dataset from the Aave V2 protocol was structured and did not contain missing values in essential fields such as transaction type, wallet address, or timestamp.

However, during the feature engineering process, certain calculated features like:

repay_ratio = total repays / total borrows

redeem_ratio = total redeems / total deposits

borrow_deposit_ratio = total borrows / total deposits

led to NaN values in cases where the denominator was zero (e.g., no borrow or deposit activity).

To ensure scoring integrity:

All division-by-zero cases were handled using conditional logic.

For example, if a wallet had zero borrows, the repay_ratio was set to 0.

Similarly, if a wallet had no deposits, the redeem_ratio was set to 0.

Wallets with extremely low or bot-like activity (e.g., total transactions = 3) were assigned the lowest score range to reflect minimal participation.

Thus, no nulls remained in the final dataset used for scoring.

##  Feature Engineering

For each wallet, we compute:

### Basic Activity Features:
- ‘total_tx`: Total number of transactions
- `total_amount`: Sum of all transaction amounts
- `deposits`, `borrows`, `repays`, `redeems`, `liquidations`: Count of each action type

### Behavior-Based Ratios:
- `repay_ratio = repays / (borrows + 1)`
- `borrow_deposit_ratio = borrows / (deposits + 1)`
- `redeem_ratio = redeems / (deposits + 1)`

### Risk Flags:
- `liquidation_flag = 1` if wallet had any `liquidationcall`, else `0`

---

##  Scoring Logic

Each wallet is assigned a raw score using:
raw_score = (
0.25 * total_tx +
0.25 * total_amount +
0.15 * repay_ratio +
0.15 * (1 - liquidation_flag) +
0.10 * borrow_deposit_ratio +
0.10 * (1 - redeem_ratio)
)

Then scaled using MinMaxScaler to bring the final `score` in the range 0 to 1000.

---

##  Score Distribution

Wallet scores are binned into the following ranges:

- (0–100]
- (100–200]
- (200–300]
- (300–400]
- (400–500]
- (500–600]
- (600–700]
- (700–800]
- (800–900]
- (900–1000]

Score Distribution Graph

![Score Distribution](https://github.com/sanjaystats7057/aave-wallet-credit-score/blob/main/score_distribution.png)


---

##  How to Run

### Requirements
- Python 3.8+
- pandas, numpy, matplotlib, scikit-learn

### Run the script:

```bash
python score_wallets.py sample.json
This will:
•	Load and preprocess the JSON file
•	Aggregate transactions per wallet
•	Compute behavioral features
•	Assign a credit score
•	Output a final CSV with scores
Files Included
•	score_wallets.py: Main script to read JSON, score wallets
•	analysis.md: Behavioral analysis and interpretation
•	score_distribution.png: Score histogram
•	README.md: Project explanation
________________________________________
 Insights
•	Low score wallets (0–300): Usually bots, single-use wallets, or those with liquidation history and poor repayment.
•	High score wallets (700–1000): Consistently active, repay loans, avoid liquidation, behave like responsible DeFi users.


