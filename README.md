
---

![Thumbnail](https://i.postimg.cc/nrnmnwfh/Gemini-Generated-Image-7mnqjr7mnqjr7mnq-(1).png)

---
## **🎯 Problem Definition**

**This project focuses on building a predictive model to estimate the target variable based on key trip-related features.** 
The goal is to create a reliable, interpretable model that can generalize well to unseen data. The workflow covers the entire pipeline from raw data to a trained model ready for use.

---
## **📥 Data Collection**

The dataset used in this project originates from "Kaggle" (A well known website for public datasets). It includes information such as passenger counts, trip cost, distances, and timestamps.
The raw data was imported in its original form to ensure reproducibility and transparency.

Link to mentioned "Kaggle" dataset: `https://www.kaggle.com/datasets/yasserh/uber-fares-dataset`


#### 🗂️Portion of the raw data:

| `Unnamed`: 0 | `key`                         | `fare_amount` | `pickup_datetime`       | `pickup_longitude` | `pickup_latitude` | `dropoff_longitude` | `dropoff_latitude` | `passenger_count` |
| ------------ | ----------------------------- | ------------- | ----------------------- | ------------------ | ----------------- | ------------------- | ------------------ | ----------------- |
| 24238194     | 2015-05-07 19:52:06.0000003   | 7.5           | 2015-05-07 19:52:06 UTC | -73.999817         | 40.738354         | -73.999512          | 40.723217          | 1                 |
| 27835199     | 2009-07-17 20:04:56.0000002   | 7.7           | 2009-07-17 20:04:56 UTC | -73.994355         | 40.728225         | -73.994710          | 40.750325          | 1                 |
| 44984355     | 2009-08-24 21:45:00.00000061  | 12.9          | 2009-08-24 21:45:00 UTC | -74.005043         | 40.740770         | -73.962565          | 40.772647          | 1                 |
| 25894730     | 2009-06-26 08:22:21.0000001   | 5.3           | 2009-06-26 08:22:21 UTC | -73.976124         | 40.790844         | -73.965316          | 40.803349          | 3                 |
| 17610152     | 2014-08-28 17:47:00.000000188 | 16.0          | 2014-08-28 17:47:00 UTC | -73.925023         | 40.744085         | -73.973082          | 40.761247          | 5                 |

---
## **🧹 Data Cleaning**

The initial dataset required preparation before analysis. This included handling missing values, correcting inconsistent entries, removing invalid or extreme outliers, and standardizing formats. These steps ensured the data was accurate, internally consistent, and suitable for model training.

#### 🧼Stage One:

#### Processing Steps:
	
- **Removing irrelevant information:** With removing information columns such as "Unnamed: 0" and "key" we rid ourselves of useless data and speed up the processing time.
	
- **Transforming longitudes to actual distances:** 
		**Distance Calculation Method:**
		To calculate the distance between `pickup` and `dropoff` coordinates, we use the **Haversine Formula**. This accounts for the curvature of the Earth.
		Where: $\phi$ is latitude, $\lambda$ is longitude (converted to radians) and $R$ is the Earth's mean radius ($6,371 \text{ km}$).
	
- **Cleaning Pickup time:** The importance of pickup time in our data is for the hour of travel; thus we delete the irrelevant information from the column.


$$a = \sin^2\left(\frac{\Delta\phi}{2}\right) + \cos(\phi_1) \cdot \cos(\phi_2) \cdot \sin^2\left(\frac{\Delta\lambda}{2}\right)$$
######
$$Distance = 2 \cdot R \cdot \ atan2\left(\sqrt{a}, \sqrt{1-a}\right)$$


#### 🗂️Portion of the cleaned data:

| `fare_amount` | `passenger_count` | `distance_m` | `pickup_time` |
| ------------- | ----------------- | ------------ | ------------- |
| 7.5           | 1                 | 1683         | 19            |
| 7.7           | 1                 | 2457         | 20            |
| 12.9          | 1                 | 5036         | 21            |
| 5.3           | 3                 | 1661         | 08            |
| 16.0          | 5                 | 4457         | 17            |

###### Visualization of _"Stage One"_ Cleaned Data:
![Cleaning Scatter](https://i.postimg.cc/dVVzhqjh/1-raw-scatter-plots-plotly.png)


#

#### 🛠️Stage Two:

#### Sanitizing Steps:
Our data is most likely hosting some problematic rows among our dataset. It may be consisted of missing values, outlier values, etc.
We use "Sanitizing Methods" to ensure the clearage of our dataset.
	
### 1️⃣Method One: Logical Sanitization
1. We remove the rows consisting of distance values below 0
2. Making sure that at least 80m per dollar has been spent by removing inappropriate rows.
3. Removing the rows to ensure not more than 500m per dollar has been registered.
4. Making sure passenger numbers are between 0 and 10 by removing the outliers.

###### Visualization of Sanitized Data with _Method One_:
![Method One Sanitizing Scatter](https://i.postimg.cc/xdW7pLDW/2-sanetized-method-1-scatter-plots-plotly.png)




### 2️⃣Method Two: IQR (Interquartile Range)
The **Interquartile Range (IQR)** is a measure of **statistical dispersion** widely used in data sanitization (cleaning) and exploratory data analysis, particularly for **detecting and handling outliers**. It's considered a **robust measure** because it's based on the middle 50% of the data, making it less sensitive to extreme values (outliers) than the total range or standard deviation.

#### Defining the IQR
The IQR is simply the difference between the **Third Quartile ($Q_3$)** and the **First Quartile ($Q_1$)** of a dataset:
	
$$\text{IQR} = Q_3 - Q_1$$
	
* **First Quartile ($Q_1$):** The value below which the lowest **25%** of the data lies (the 25th percentile). It is the median of the lower half of the data.
* **Third Quartile ($Q_3$):** The value below which the lowest **75%** of the data lies (the 75th percentile). It is the median of the upper half of the data.
* **IQR:** Represents the range or spread of the central **50%** of the data.

#
#### Using IQR for Outlier Detection (Data Sanitizing)
The most common application of the IQR in data sanitizing is to define a "fence" or range to identify potential **outliers** using the **$1.5 \times \text{IQR}$ Rule**. Any data point falling outside this range is flagged as an outlier and may be removed or investigated.

#### 1. Calculate the Fence
The outlier boundaries (or "fences") are calculated using the following formulas:
* **Lower Bound:** $Q_1 - 1.5 \times \text{IQR}$
* **Upper Bound:** $Q_3 + 1.5 \times \text{IQR}$

#### 2. Identify and Sanitize Outliers
* A data point is considered a **low outlier** if its value is **less than** the **Lower Bound**.
* A data point is considered a **high outlier** if its value is **greater than** the **Upper Bound**.


###### Visualization of the Sanitized data with _Method Two_:
![Method Two Sanitizing Scatter](https://i.postimg.cc/d1rfbDwg/3-sanetized-method-2-scatter-plots-plotly.png)
#### **_As you can observe here and we will discuss the reasons later on, this sanitizing method does not perform well at all on this dataset!_**
1. Relevant and good data has been removed by this method.
2. Estimate of 3K rows of data has been removed.
3. Data's range has been considerably lowered.
---
## **⚙️ Feature Engineering**

### Time-based Values:
Time-based columns were transformed into cyclic components:

#### Sine and Cosine Transformation:
This method transforms the single time feature into two new features that capture the cyclical nature of the data. It uses trigonometric functions to map the time onto a circle, where the start and end points meet.

Assuming our time is currently in hours, $H$, where $0 \le H < 24$, and considering the period ($P$) for time-of-day to be 24 hours, we calculate the Sine Component ($\mathbf{H_{sin}}$): 
This captures the vertical (or North-South) component on the circle:
$$H_{sin} = \sin \left( \frac{2\pi H}{P} \right) = \sin \left( \frac{2\pi H}{24} \right)$$

Calculate the Cosine Component ($\mathbf{H_{cos}}$): This captures the horizontal (or East-West) component on the circle:
$$H_{cos} = \cos \left( \frac{2\pi H}{P} \right) = \cos \left( \frac{2\pi H}{24} \right)$$


#
### Numerical Values:
Numerical values were normalized where appropriate, and only the most informative variables were retained. 
#### Min-Max Scaling (Normalization):
This is the standard method for strictly transforming data to the closed interval $[0, 1]$.
**Use case:** When the model requires input values to be strictly non-negative and within a fixed range (e.g., neural networks often prefer inputs between 0 and 1).
**Sensitivity:** High sensitivity to outliers, which can compress the bulk of the data into a small portion of the $[0, 1]$ range.

**Formula:** $$x_{scaled} = \frac{x - \text{min}(x)}{\text{max}(x) - \text{min}(x)}$$


##### Standardization (Z-Score Scaling):
This method transforms the data to have a mean of 0 and a standard deviation of 1.
**Use case:** When you suspect your data contains outliers or when the algorithm assumes a Gaussian (Normal) distribution (e.g., Linear Regression, k-Nearest Neighbors).
Range: The resulting range is theoretically $(-\infty, \infty)$, but the vast majority of values fall between $\approx -3$ and $\approx 3$.

**Formula:** $$x_{scaled} = \frac{x - \mu}{\sigma}$$$\mu$ is the mean of the feature values.$\sigma$ is the standard deviation of the feature values.
###### **_After thorough observation and testing, this normalization method does not perform well compared to "Min-Max Scaling" on this dataset! thus it will not be used._**



### 🗂️Portion of the engineered data:

| `fare_amount` | `passenger_count` | `distance_m` | `h_sin`   | `h_cos`       |
| ------------- | ----------------- | ------------ | --------- | ------------- |
| 0.032567      | 0.166667          | 0.027626     | -0.965926 | 0.258819      |
| 0.033436      | 0.166667          | 0.040346     | -0.866025 | 0.500000      |
| 0.056046      | 0.166667          | 0.082729     | -0.707107 | 0.707107      |
| 0.023001      | 0.500000          | 0.027264     | 0.866025  | -0.500000     |
| 0.069525      | 0.833333          | 0.073510     | -0.965926 | -0.258819<br> |
###### These engineered features were used as the model inputs.

---
## **📊 Model Selection and Training**

![Linear Regression](https://i.postimg.cc/MHzBzrJc/Gemini-Generated-Image-8w80wh8w80wh8w80.png)

A linear model was selected for this project due to its suitability for the structure of the available data and the objectives of the analysis. The primary features used for prediction, including `distance`, `passenger count`, and the cyclic time components derived from the timestamp (`h_sin` and `h_cos`), are all variables that are expected to influence the target in a smooth and approximately linear manner. This makes linear regression an appropriate first choice for modeling their relationships.

Another factor supporting this selection is interpretability. Linear models provide clear, direct insight into the contribution of each feature, allowing the model’s behavior to be understood and communicated effectively. This transparency is valuable both for validating design choices and for explaining the system to non-technical audiences.

From a practical standpoint, linear regression is computationally efficient and performs well on datasets of moderate size without the need for extensive hyperparameter tuning. Combined with the carefully engineered features, especially the cyclic time representation that linearizes periodic patterns, the model is able to capture the essential structure of the data without unnecessary complexity.

Overall, a linear model offers a strong balance of performance, simplicity, and interpretability, making it a fitting choice for this predictive task.

---
## **🖼️ Evaluation & Visualization**
The trained model was evaluated using standard regression metrics. These scores reflect how well the model captures overall trends and how accurately it predicts unseen samples.


#### **✅Results Of Method One: (Logical Sanitization)**
![Method one result](https://i.postimg.cc/J0RXRdCj/Gemini-Generated-Image-12bg5u12bg5u12bg.png)
#### 📄Model Performance Explanation:

##### Mean Squared Error (MSE): 0.000217
This value represents the average squared difference between the model’s predictions and the actual target values.
A lower MSE indicates that the model’s predictions are very close to the true outcomes.
Given the scale of your normalized data, an MSE around 2.17e-4 is very low, showing that the model captures the underlying pattern well and does not produce large errors.
##
##### R² Score: 0.8741
The R² score (coefficient of determination) measures how much of the variance in the target variable is explained by the model.
An R² of 0.874 means the model explains 87% of the variability in the output.

In practical terms:
Our linear model is doing a strong job, and most of the target’s behavior is successfully captured by the features `['passenger_count', 'distance_m', 'h_sin', 'h_cos']`.

##
##### Coefficients
Our learned coefficients were:
```css
[ 0.00091027   0.66516669  -0.00122833  -0.00331209 ]
```
Each coefficient represents how much the predicted value changes when that feature increases by one unit (with other features held constant).

Interpreting them:
- _`passenger_count`_: 0.00091
	A small positive effect. More passengers slightly increase the predicted value.
	
- _`distance_m`_: 0.66517
	This is the dominant driver. Distance strongly contributes to the final prediction, which matches real-world intuition.
	
- _`h_sin`_: –0.00123 and _`h_cos`_: –0.00331
	Time-of-day components (sin/cos) have mild negative effects.
	They don’t dominate, but the model still finds a subtle pattern in how time influences the target.

**Intercept: 0.01211**
This is the baseline predicted value when all input features are zero.
Since our inputs are normalized, this isn’t meant to be interpreted in isolation; it just centers the model correctly.

###### 📈 Method One Model Result: (Logical Sanitization)
![Result of Method One Scatter](https://i.postimg.cc/jq69vW8N/4-result-nor1-scatter-plots-plotly.png)




#### **❌Results Of Method Two: (IQR)**
![Method two result](https://i.postimg.cc/T1xVp3gW/Gemini-Generated-Image-kea8blkea8blkea8.png)

#### 🆚Comparison of Scaling Methods: Min-Max Normalization vs. IQR Scaling
1. Min-Max Normalization Results (First Model)
	MSE: 0.000217
	R²: 0.8741
	Coefficients: [0.00091027, 0.66516669, -0.00122833, -0.00331209]
	Intercept: 0.01211

**Interpretation**:
This model performs well. The distance feature retains a strong, stable coefficient (~0.665), indicating that the relationship between distance and the target variable remains clean and consistent after scaling.
The time-based sinusoidal features and passenger count contribute smaller but meaningful refinements.
Overall, the model explains about 87% of the variance, which is very strong for a linear model on real-world transportation data.

##

2. IQR (Interquartile Range) Scaled Results (Second Model)
	MSE: 0.0015598
	R²: 0.6941
	Coefficients: [0.00296575, 0.28781381, -0.0038285, -0.00914904]
	Intercept: 0.04924

**Interpretation**:
After applying IQR scaling, the model’s performance decreases significantly.
The R² drops from 0.874 to 0.694, meaning the model now explains only 69% of the variance instead of 87%.
Distance, previously the dominant predictor, now has a coefficient of about 0.2878, indicating that the scaling distorted the relative magnitude and distribution of this feature.

###### 📉 Method Two Model Result: (IQR)
![Result of Method Two Scatter](https://i.postimg.cc/L6HWWH7W/5-result-nor2-scatter-plots-plotly.png)


### **❓Why IQR Scaling Is Not Suitable for This Dataset**

#### IQR scaling is designed for extreme outlier resistance
IQR works by subtracting the median and dividing by the interquartile range (Q3–Q1).
This is useful when a feature has extreme, irregular outliers that we want the model to completely ignore.

But in our dataset:
- We already cleaned outliers manually.
- The remaining values, especially distance and time, are meaningful and smoothly distributed.
- IQR shrinks the natural spread of features too aggressively.

The result is a squashed, distorted representation of the data.

##

#### Distance (your most important feature) gets flattened
Distance has a wide, continuous distribution that carries the majority of the predictive signal.

With IQR scaling:
- The middle 50 percent gets squeezed tightly.
- The outer areas (which still contain meaningful long trips) get compressed.
- The model loses contrast between trips of different lengths.

This is why the distance coefficient falls from 0.665 → 0.288.
Our model becomes less sensitive to actual differences in trip length, so accuracy takes the impact.

##

#### Time features are already bounded
Despite not applying this method on time values; our `h_sin` and `h_cos` features already live in [–1, 1].

Applying IQR scaling on top of a bounded, circular feature:
- Adds inconsistency.
- Breaks symmetric relationships.
- Introduces no real benefit.

These features are already perfect as they are.

##

#### Min-max fits our dataset’s nature
Min-max scaling preserves:
- Feature shape
- Relative differences
- Original structure of the distribution

It makes all features comparable without erasing the underlying meaning.
That’s exactly what benefits a linear model.

Our first model proved the explanation: High R², low MSE, stable coefficient behavior.

##

#### 💡Conclusion
IQR scaling degraded model quality because it over-compressed features that contained crucial predictive structure, especially distance.
Min-max scaling preserved the natural relationships in the data, producing a far more accurate and stable model.

---
## **📦 Model Saving and Reusability**

The final trained models was serialized and saved using a standard format (Pickle). This allows the model to be loaded and used for inference without requiring the full training code.
- The model trained with "**Method One**" normalization: `models\Uber_Linear_Model_Ver2.pkl`
- The model trained with "**Method One**" & "**Method Two**" normalization: `models\Uber_Linear_Model_Ver1.pkl`
- Plot images are available with two variants for each at: `results\figures\`
- Raw data can be found at: `data\raw`
- Processed data can be found at: `data\processed`
- Test values can be found at: `data\test`

---

