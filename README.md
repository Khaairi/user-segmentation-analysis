# User Segmentation Analysis
## Overview
This project focuses on a comprehensive user segmentation approach that bridges the gap between raw transactional data and actionable business personas. In many real-world scenarios, direct demographic data (like age or income) is unavailable. This project addresses heuristic-based Feature Engineering to infer user profiles and combining them with RFM (Recency, Frequency, Monetary) analysis to create a 360-degree view of the customer base.

## Dataset
The analysis is performed on a rich transactional dataset containing over **250.000** data with the following attributes:
- `transaction_id`: Unique identifier for each transaction.
- `user_id`: Unique identifier for each user.
- `transaction_amount`: The amount of money transacted.
- `transaction_date`: The date and time when the transaction occured.
- `merchant_id`: Unique identifier for each merchant.
- `merchant_category_id`: Identifier for the category of the merchant (Visa MCC category ID).
- `geo_location`: The geographical location of the transaction (latitude and longitude).
- `payment_method`: Method of payment used (balance, credit_card).
- `user_agent`: Type of device used for the transaction.
- `loyalty_program`: Indicates whether the user is part of a loyalty program (yes, no).
- `discount_applied`: Indicates whether a discount was applied to the transaction (yes, no).
- `promo_amount`: The amount discounted due to promotions.
- `transaction_notes`: Additional notes or descriptions related to the transaction.
- `merchant_rating`: User's rating of the merchant (on a scale of 1-5).
- `transaction_status`: Status of the transaction.
- `is_refunded`: Indicates whether the transaction was refunded.

## Feature Engineering
To quantify demographics without direct identifiers, I developed a weighted scoring model using various behavioral proxies:
### **Age Feature Engineering Methodology**
<img width="584" height="384" alt="age_feat" src="https://github.com/user-attachments/assets/be493ad2-1fa0-4f5a-85db-786363668d9f" />

In the absence of direct demographic identifiers regarding age, I developed a weighted scoring model to quantify user age groups. This methodology treats the hardware lifecycle, consumption patterns, and purchasing power as digital proxies for age.
- The model calculates the degree of device "newness" through the detected operating system version from the `device_category`. Users with the latest OS (such as Android 14 or iOS 17) are classified as "Early Tech Adopters," a segment statistically dominated by younger age groups. Conversely, the use of older (Legacy) OS versions often correlates with older age groups or individuals with slower device replacement cycles.
- I assigned "Age Points" to specific Merchant Category Codes (MCCs) based on lifestyle relevance and life-stage priorities. Categories such as Fast Food (MCC 5814) and Video Game Establishments (MCC 7994) are weighted toward the "Youth/Student" profile. In contrast, spending on Pharmacies (MCC 5912), Furniture Stores (MCC 5712), or Travel Agencies (MCC 4722) is weighted more heavily toward the "Mature" or "Senior" profile, reflecting typical household and health-related spending patterns.
- I integrated the average spending per transaction as a secondary signal. This approach is grounded in economic trends where disposable income and average transaction value tend to peak as individuals advance in their careers and reach maturity, typically peaking within the "Established Adult" demographic.
- By averaging these multidimensional scores per user, the model places individuals into four distinct age bins: **Gen Z / Student**, **Young Professional**, **Established Adult**, **Senior**.

### **Gender Feature Engineering Methodology**
<img width="584" height="384" alt="gender_feat" src="https://github.com/user-attachments/assets/4314230a-9e03-4a04-b81c-e9cd9167aba7" />

In the absence of ground-truth demographic regarding gender, I developed a Probabilistic Scoring Model that utilizes Merchant Category Affinity as the primary signal for gender inference, supplemented by behavioral weighting based on promotion sensitivity.
- This model assigns "Gender Points" to specific Merchant Category Codes (MCCs) that statistically exhibit a strong gender bias. For instance, transactions in Beauty & Barber Shops (MCC 7230) or Cosmetics Stores are weighted toward feminine scores, while transactions in Hardware Stores (MCC 5251) or Automotive Parts are weighted toward masculine scores. This approach acknowledges that while categories are not exclusive, they serve as high-probability behavioral indicators.
- Grounded in consumer behavior research indicating differences in sensitivity to price incentives, this model integrates the `discount_applied` feature. Higher coupon usage frequency and a higher response rate to promotional offers are used as weighted factors to identify shopping profiles traditionally more dominant in the female segment, which often exhibits a stronger orientation toward coupons and discount-driven shopping. [Men Vs Women: online shopping habits](https://www.paymentsense.com/uk/blog/men-vs-women-online/#:~:text=2.,SALE%20signs%20in%20shop%20fronts.%5D)
- To acknowledge the inherent uncertainty in behavioral proxies, I chose a probabilistic scoring method rather than a rigid binary classification. All signals are processed through aggregation per `user_id` to create a weighted average value. This ensures that a single outlier transaction does not skew the entire profile.
- The final step involves binning these continuous probability scores into three distinct categories: **Likely Male**, **Likely Female**, **Neutral / Undetermined (for users with highly balanced or insufficient behavioral signals)**.

### **Income Level Feature Engineering Methodology**
<img width="584" height="384" alt="income_feat" src="https://github.com/user-attachments/assets/4735b40d-b59c-4c62-bd43-fb39c5b57aec" />

To estimate user economic stratification without access to official income data, I developed a weighted scoring model that utilizes transaction context, device asset ownership, and promotion sensitivity as high-fidelity proxies for income level.
- Rather than simply evaluating the absolute value of a transaction, the model calculates the degree of deviation from the average spend within specific merchant categories. For example, a Rp 2,000,000 transaction at a Fast Food restaurant (MCC 5814) is flagged as an indicator of very high disposable income, whereas the same amount spent at a Travel Agency (MCC 4722) is classified as typical middle-class spending behavior.
- The model maps the device hierarchy using `device_category` and OS versions to infer purchasing power. The highest weights are assigned to the Apple ecosystem (iPhone/iPad), particularly those running the latest OS versions (v13+), as indicators of premium purchasing power. Flagship Android devices (v11+) occupy the upper-middle position, while devices with single-digit OS versions (Legacy) serve as a proxy for lower economic segments or users with low device-update priority.
- I established an "Income Points" system based on the prestige and cost-barrier of merchant categories. High-barrier categories such as Airlines, Luxury Retail, and Car Dealers receive maximum scores, while Basic Needs categories (e.g., local groceries) serve as the baseline for lower-middle income levels.
- The model analyzes the ratio of promotional usage (`promo_amount`) relative to total spending. A low reliance on discounts, coupled with active participation in loyalty programs, is used to identify high-income users who prioritize convenience, brand exclusivity, and time-efficiency over price sensitivity.
- All the variables are aggregated per `user_id` using a weighted average. This final score is then binned into three primary economic categories: **Low Income**, **Middle Income**, **High Income**.

### **Working Status Feature Engineering Methodology**
<img width="584" height="384" alt="work_feat" src="https://github.com/user-attachments/assets/cfae57a8-c5df-4a37-a4ea-8b5a69b60314" />

In the absence of direct working status labels, I designed an inference model based on Temporal-Spatial Routine and Financial Responsibility to differentiate users based on their professional rhythm and financial stability.
- This model analyzes the distribution of transaction timestamps to distinguish schedule flexibility. Users who concentrate their non-essential transactions (excluding routine necessities like service stations or fast food) on weekends are identified as Full-time Professionals with rigid weekday schedules. Conversely, shopping activity that is evenly distributed throughout the weekdays indicates the time flexibility typically enjoyed by Freelancers or Student/Unemployed individuals.
- The model utilizes the chosen payment method as a signal of income stability. Credit card usage is given a higher weight for identifying permanent workers or established freelancers, as ownership of these instruments typically requires verification of stable cash flow and creditworthiness, a contrast to the lower barrier of entry for digital balances (e-wallets).
- I identified financial responsibility by tracking transactions in education-related categories, such as Colleges (MCC 8220) or Elementary Schools (MCC 8211). Large-volume transactions in these categories are interpreted as indicators of a head of household or an individual in an active employment stage (Working Adults), regardless of whether they are permanent or freelance.
- Total expenditure volume serves as the final refinement filter. High and cyclically stable spending capacity, strengthens the classification of permanent workers. Meanwhile, medium to low spending with irregular, non-cyclical patterns is more likely to indicate students or individuals without permanent employment.
- All variables, financial instrument choices, and category-specific patterns are aggregated using a weighted average. This score is then binned to place users into one of three categories: **Full-time Professional**, **Freelancer / Flexible Worker**, **Student / Unemployed**

## Segmentation
The integration of RFM (Recency, Frequency, Monetary) metrics with inferred demographic features (Age, Income, Gender, Working Status) creates a 360-degree Customer View. While RFM is excellent for understanding what happened (behavioral history), it often lacks the context of who is performing the action. For instance, two users might have identical high-monetary scores, but one could be a High-Income Young Professional while the other is an Established Adult Full-time Worker. By combining these dimensions, we can differentiate marketing messages based on life stages and economic power, even for users with similar spending habits.
### **Segmentation Pipeline**
I utilized an unsupervised machine learning workflow to identify natural groupings within the enriched dataset:
1. Successful transactions were aggregated per `user_id` to calculate the three RFM pillars. These were then merged with the categorical scores derived from the Feature Engineering stage.
2. Financial data like `Monetary` and `Frequency` often follow a power-law distribution. I applied `np.log1p` to these features to normalize the distribution, ensuring that extreme "whales" (outliers) do not disproportionately bias the clustering algorithm.
3. Because features exist on different scales (e.g., Monetary in millions vs. Age Score from 1-4), I applied `StandardScaler`. This ensures each feature contributes equally to the distance calculations in the clustering process, preventing large-numerical features from dominating.
4. I employed the K-Means Clustering algorithm. To determine the most effective number of clusters ($k$), I utilized the Elbow Method, identifying the point where increasing $k$ yields diminishing returns in variance explanation.

## Results and Profiling
<img width="475" height="363" alt="pca" src="https://github.com/user-attachments/assets/04454da9-3b2c-4716-9d23-897707c5cb0d" />

The model identified 6 distinct segments with unique characteristics:
- **Cluster 0: The Most Active Mid-Market (Most Valuable)**

  They have the lowest Recency (3.1 days), meaning they are the most recent and frequent transacting group. Their Frequency (12.8) and Monetary are the highest among all clusters. This is the "heart" segment of the business. Despite their freelance status, they are highly active and have significant expenses.
- **Cluster 1: The High-Income Young Elite**

  Has high Monetary even though the number of users is the smallest (exclusive group). They have high purchasing power but may be more selective or transact less frequently than Cluster 0.
- **Cluster 2: The Steady Mid-StreamERS**

  It is at the mid-level for all RFM metrics. A group of young professionals who are building their careers. They are the future of Cluster 4 or Cluster 1.
- **Cluster 3: The Low-Engagement Segment**

  It has the smallest Frequency (7.2) and Monetary values among all clusters. Its recency is also quite long (13.5 days). The least active segment. They may just be trying out the service or only showing up when there's a big promotion that's highly relevant to them.
- **Cluster 4: The Loyal Professionals**

  Similar to Cluster 0 in terms of Frequency (12.5) and Monetary, but slightly less recent in terms of transactions (Recency 8.2 days). This group consists of established office workers. They tend to have stable and predictable spending patterns.
- **Cluster 5: The Occasional Established Buyers**

  They have the highest Recency (14.3 days), meaning they haven't made a transaction in a while. However, when they do make a purchase, their transaction value is quite large. Established customers who shop only when needed (utility shoppers).
