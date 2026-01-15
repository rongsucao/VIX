### **1. What is VIX**   
Definition:    
The VIX Index (CBOE Volatility Index) measures the market's expectation of 30-day implied volatility for the S&P 500 Index. It is calculated from a weighted strip of out-of-the-money (OTM) SPX options.    
<img width="281" height="55" alt="image" src="https://github.com/user-attachments/assets/3d9cdbf1-2916-4d8c-9b57-ba12dfc8e8c0" />   


Key Characteristics   
(1) Unit  
Annualized implied volatility (percentage points)   
(2) Underlying  
S&P 500 Index (SPX) options  
(3) Horizon  
30-day forward-looking  

Interpreting VIX Levels:  
> VIX = 20 means the market expects SPX to move ±20% over one year (annualized)   

**Converting to Different Time Horizons:**  

![alt text](image.png)  
**Example:**  
VIX = 16  
Expected monthly move: 16/3.46 ≈ 4.6%  
Expected daily move = 16/16 = 1%  

### **2. Understanding the Data Structure**  

| Column    | Meaning | Example |
| -------- | ------- |-------|
| Date  | Valuation date   | 2001-06-11|
| Option Expiry | Option expiration date     | Various dates|
| Option Type    | Put/Call    | P or C|
|Strike | Strike Price | 1100,1250, etc |
|Option Price|Market price of option|Varies|
|Forward| Forward Price| 1254.45|
|Discount Rate|Risk-free Rate| Small Value|
|Implied Volatility| Backed-out IV| 0.177|
|Spot|Current Spot Price| 1254.39|   

**OTM option selection**  
Suppose forward = 1254.45  
For strikes < forward: we use OTM Put option.  
For strikes > forward: we use OTM Call option.  

<img width="715" height="115" alt="image" src="https://github.com/user-attachments/assets/0b32d9e1-846c-42be-b4d8-648947991ce9" />

**Why compare strike price with Forward Price?**  
Key reason：options payoff happens in strike date instead of today.  
When we ask, if this option is in the money or out of the money, the real question is :  
> Where will the underlying be relative to the strike at maturity?  

Mathematical Explanation: Risk-Neutral Measure   
Under black-scholes framework, the expected value of underlying price under the risk neutral measure is:  
<img width="235" height="31" alt="image" src="https://github.com/user-attachments/assets/8711af11-c6b0-4a4a-8c17-a0accf4ca4fb" />  
This is the forward price.  
Therefore when we define OTM/ITM, we should use the market's best estimate of the practice at expiration, which is the Forward.  
(In the risk-neutral world, all assets are expected to grow at the risk-free rate.)  

**WHy does low-strike IV > high-strike IV?**  
for example:  
strike 1100: IV = 42.5%  
strike 1250: IV = 17.7%  
This phenomenon is called the Volatility Skew.  

Why does the skew exist?  
|Reason|Explanation|  
|---|---|  
|Crash Fear| Investors fear downside more than upside|  
|Demand Imbalance| Heavy demand for OTM puts(portfolio protection)|  
|Supply Constraints| Dealers charge premium for braring tail risk|  
|Empirical Reality| Market crash fast, rally slow|    

The volatility skew reflects the market's asymmetric fear. Investors are willing to pay more for protection against large downward moves than for upside exposure.  

**Why does CBOE use only OTM options?**  
There are 3 reasons for CBOE only use OTM options?  
  
Reason 1: OTM options provide purer volatility information.  
> OTM Option Value = Pure Time Value = f(volatility, time)  
> ITM Option Value = Intrinsic Value + Time Value  

Reason 2: Put-Call Parity  
ITM put option and OTM Call option contain the same volatility information.  
<img width="251" height="51" alt="image" src="https://github.com/user-attachments/assets/0862dc41-789c-4252-91e6-3ec894777ccc" />  
The difference is deterministic (no vol component), so using both would be redundant.  

Reason 3: Better Liquidity  
OTM options are more actively traded → more reliable prices → better VIX calculation.  

### **3. VIX Calculation: The CBOE Methodology**   
Formula:  
<img width="404" height="63" alt="image" src="https://github.com/user-attachments/assets/007819b7-4507-4683-af47-7a68785524da" />  
Where:  
- T = Time to expiration(30 days/ 365)
- $K_i$ = Strike price of the i-th OTM option
- $ΔK_i$ = Interval between strike prices
- $Q(K_i)$ = Midpoint of bid-ask spread for oprion with strike $K_i$
- F = Forward index level
- $K_0$ = First strike below the forward price

|Component|Symbol|Meaning|  
|--|--|--|  
|Option price| $Q(K_i)$ | Price of the option. The higher the price of the option, the higher possibility market think stock could achive strick price, the greater the contribution.|  
|Strike Spacing| $ΔK_i$ | Spacing. Represents how wide of a strike range this specific option is "responsible" for covering|
|Weight|$1/K^2_i$|Adjustment Factor. Options with lower strikes are assigned a greater weight (inverse square weighting).|
|Discounted Factor|$e^{rT}$|Interest Rate Adjustment. Usually very small and can typically be ignored.|  
|Adjustment Term|$\frac{1}{T}\left(\frac{F}{K_0} - 1\right)^2$|Technical Adjustment Term. Corrects for the difference between the Forward price ($F$) and the cutoff strike ($K_0)$|


**Calculate Variance Swap Fair Strike for a Single Expiry**  
A variance swap can be replicated by a portfolio of options.  
> Variance Swap Fair Strike = Cost of replicating option portfolio.  
> We can back out the market expected variance from option prices.

**Intuition: Why $1/K^2$ Weighting**  
Why low strike options have higher weights?  
Answer: To make sure we have constant dollar gamma across all spot prices.  

Volatility is about percentage returns, not absolute dollar moves.  
A $10 move when the stock is at $100 is a huge volatility event (10%), but the same $10 move at $1000 is nothing (1%).  
The $1/K^2$ weighting simply adjusts for this, giving more importance to the lower strikes because price changes down there represent much larger percentage swings.    

The VIX is designed to measure the variance of Log Returns ($\ln \frac{S_T}{S_0}$), because investors care about percentage moves, not absolute dollar moves.  
Therefore, the target payoff we are trying to synthesize (replicate) is a Logarithmic Curve: $f(S) = \ln(S)$.  
Options (Calls and Puts) have linear payoffs at expiration (straight lines like a hockey stick). To create a curved shape (like $\ln S$) using straight lines (Options), you need to combine a portfolio of options across all strikes.The Rule: To replicate a curved function $f(S)$, the number of options you need at a specific Strike ($K$) is equal to the second derivative (curvature) of that function.  

Let's do the calculus on our target function, the Log profile:  
Target Function: $$f(K) = \ln(K)$$  
First Derivative (Slope): $$f'(K) = \frac{1}{K}$$  
Second Derivative (Curvature/Weight): $$f''(K) = -\frac{1}{K^2}$$  

The Intuitive Summary:  
> Low Strikes ($K$ is small): The Log curve is extremely curved/steep at low numbers (e.g., the difference between $\ln(10)$ and $\ln(11)$ is much sharper than $\ln(1000)$ vs  $\ln(1001)$ ). Because the curve bends so much there, you need a massive weight ( $1/K^2$ ) of options to replicate that shape accurately.  
> High Strikes ($K$ is large): The Log curve flattens out. Since it is almost a straight line already, you need very little weight to replicate it.  

### **4.Step-by-Step Calculation**  
**Step 1.1: Identify OTM Options**  
Action: Select which options to include in the calculation.  
Logic: Look at the Forward Price(F)  
- if Strike < F: Use Put prices.
- if Strike > F: Use Call prices.
- At the center(near F): Use the averge of Put and Call prices.  
Why?
Because we only want OTM options because they are the most liquid and represent pure "volatility value".

**Step 1.2: Calculate Strike Spacing ($\Delta K_i$)**  
Action: Calculate the "width" of the strip that each option is responsible for.  
Formula: $\Delta K_i = \frac{K_{i+1} - K_{i-1}}{2}$  
Meaning:  
- This is the equivalent of $dx$ in calculus.
- If you are looking at Strike 1110, and the neighboring strikes are 1100 and 1120, the spacing is $\frac{1120 - 1100}{2} = 10$.
- If strikes are far apart, $\Delta K$ is large; if they are close together, $\Delta K$ is small.

**Step 1.3: Calculate Each Option's Contribution**  
Action: Calculate how much "variance" a single strike contributes to the total.  
Formula:   
<img width="449" height="81" alt="image" src="https://github.com/user-attachments/assets/5804a66c-6106-4429-b72e-68270f245d66" />  
Intuition: This calculates the area of the rectangle.  
- Height: The Option Price $Q(K_i)$.
- Width: The spacing $\Delta K_i$.
- Weight: The $1/K^2$ factor we discussed earlier (giving more weight to lower strikes).

**Step 1.4 & 1.5: Sum and Adjust**  
Action: Add everything up to get the total variance.  
Formula:  
<img width="381" height="88" alt="image" src="https://github.com/user-attachments/assets/b579d712-a1c2-45ad-9e32-2249c336d427" />  
Meaning:  
- Sum: Add up the contribution from every single Put and Call.
- Adjustment Term: Because the available strikes are discrete, the "ATM Strike" ($K_0$) is rarely exactly equal to the Forward Price ($F$). This term subtracts the small error caused by that difference.


> The Worked Example (Explained)
> This section walks through the math for one specific option to show how the formula works in practice.
> Scenario:
> Target: Option 1 (Strike $K = 1200$ Put).
> Inputs:  
> Spacing ($\Delta K$) = 25  
> Price ($Q$) = 15.50  
> Time ($T$) = 30 days  
> Interest ($r$) = 1%  
>The Calculation:It plugs these numbers into the Step 1.3 formula:  
> 1.Weighting: $\frac{25}{1200^2}$ (The width divided by the squared strike).   
> 2.Interest: $e^{rT}$ (A very small adjustment, close to 1).  
> 3.Price: $15.50$.  
> The Result: 0.000270  
> This number (0.000270) is the Variance Contribution of just that one 1200 Put option.  
> To get the final VIX, you would calculate this for every strike (1225, 1250, 1275, etc.), add them all up, and then take the square root.  

### **5. Interpolation to Exactly 30 Days.**  
Problem:      
The VIX definition is the 30-day expected variance. However, it is almost impossible to have an option chain that expires in exactly 30 days! From your data, there are 6 different expiry dates:    
<img width="564" height="200" alt="image" src="https://github.com/user-attachments/assets/d25e5037-c1e3-4447-85c9-993f61937c33" />    
Question: How do we get exactly 30 days of variance from this data?    
Answer: Interpolation!   

**The Critical Rule**  
We interpolate variance, not volatility.  

Why Variance, Not Volatility?  
Mathematical Reason: Variance is Additive Over Time   
$$\text{Var}(R_{t_1 + t_2}) = \text{Var}(R_{t_1}) + \text{Var}(R_{t_2})$$    
This means:  
$$\sigma^2_{T_1} \cdot T_1 + \sigma^2_{T_2} \cdot T_2 = \sigma^2_{T_1 + T_2} \cdot (T_1 + T_2)$$  
Therefore, Total Variance ($= \sigma^2 \times T$) can be directly summed and interpolated.  
Volatility Does NOT Scale Linearly!  
$$\sigma_{T_1} + \sigma_{T_2} \neq \sigma_{T_1 + T_2} \quad \text{❌ WRONG!}$$  
Volatility scales with the square root of time ($\sqrt{T}$), so it is not linear.   

**The Interpolation Formula**  
Scenario: Suppose we have the following data points:  
Near-term expiry: $T_1$ days, with annualized variance $\sigma_1^2$.   
Next-term expiry: $T_2$ days, with annualized variance $\sigma_2^2$.   
Target: $T_{30} = 30$ days.   

Step 2.1: Calculate Total Variance for Each Expiry  
First, we must "un-annualize" the variance to find the total variance for the specific time period.  
$$\text{Total Variance}_1 = \sigma_1^2 \times T_1$$  
$$\text{Total Variance}_2 = \sigma_2^2 \times T_2$$   
>Note: The value $\sigma^2$ we calculated in Step 1 was the annualized variance. However, since variance accumulates (adds up) over time, we must convert it into total variance (the actual variance expected over that specific number of days) before we can interpolate.

  
Step 2.2: Linear Interpolation in Total Variance  
Now, we draw a straight line between the two Total Variances to find the value at 30 days.  
<img width="566" height="34" alt="image" src="https://github.com/user-attachments/assets/46c8fb5e-1066-440f-b92b-d5ffa1282c46" />  
Where the time-weights are:   
<img width="275" height="56" alt="image" src="https://github.com/user-attachments/assets/47d61d65-fcb3-42a0-9926-59eaff599547" />  
Interpretation: If $T_{30}$ is closer to $T_1$, then $w_1$ will be larger. If it is closer to $T_2$, $w_2$ will be larger.   
  
Step 2.3: Convert Back to Annualized Variance  
Once we have the Total Variance for the 30-day period, we must convert it back to an annualized number.  
<img width="213" height="57" alt="image" src="https://github.com/user-attachments/assets/60cc6ab8-87b2-4140-84a4-f9d2a76594d7" />  

Step 2.4: Calculate VIX  
Finally, take the square root to get volatility, and multiply by 100 to get the index value.  
<img width="171" height="44" alt="image" src="https://github.com/user-attachments/assets/58aa9600-b6ad-4442-9589-c4d6ef4936af" />  


>Step-by-Step Calculation:
>Step 1: Convert to Total Variance Note: Time units must be consistent. Here, we use fractions of a year:  
>$$T_1 = \frac{5}{365}, \quad T_2 = \frac{40}{365}, \quad T_{30} = \frac{30}{365}$$  
>$$\text{Total Variance}_1 = \sigma_1^2 \times T_1 = 0.0324 \times \frac{5}{365} = 0.000444$$  
>$$\text{Total Variance}_2 = \sigma_2^2 \times T_2 = 0.0484 \times \frac{40}{365} = 0.005305$$  
>  
>Step 2: Calculate Interpolation Weights  
>$$w_1 = \frac{T_2 - T_{30}}{T_2 - T_1} = \frac{40 - 30}{40 - 5} = \frac{10}{35} = 0.286$$
>$$w_2 = \frac{T_{30} - T_1}{T_2 - T_1} = \frac{30 - 5}{40 - 5} = \frac{25}{35} = 0.714$$
>(Note: Since 30 days is closer to 40 days than to 5 days, $w_2$ is larger, giving more weight to the 40-day variance.)
>
>Step 3: Interpolate Total Variance  
>$$\text{Total Variance}_{30} = 0.000444 \times 0.286 + 0.005305 \times 0.714$$
>$$= 0.000127 + 0.003788 = \mathbf{0.003915}$$
>  
>Step 4: Convert to Annualized Variance  
><img width="507" height="59" alt="image" src="https://github.com/user-attachments/assets/a1291190-c68e-4539-8103-1b51e030f281" />  
>  
>Step 5: Calculate VIX  
>$$\text{VIX} = 100 \times \sqrt{0.0476} = 100 \times 0.218 = \mathbf{21.8}$$  
