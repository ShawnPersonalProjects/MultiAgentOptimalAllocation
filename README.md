One possible way to formulate and solve this kind of multi‐agent allocation (or order‐slicing) problem that balances cost minimization, trajectory constraints, and on‐the‐fly re‐evaluation of fill rates.  
  
The approach can be summarized as a multi‐stage optimization or a model‐predictive control (MPC) style procedure with stochastic elements to allow for exploration.  
  
**Notation and Setup**  
1\. Agents  
&nbsp;&nbsp;&nbsp;&nbsp;Aggressive (cost=2)  
&nbsp;&nbsp;&nbsp;&nbsp;Neutral (cost=1)  
&nbsp;&nbsp;&nbsp;&nbsp;Passive (cost=0)  

2\. Time Horizon:  
&nbsp;&nbsp;&nbsp;&nbsp;Divided into 𝐵 discrete buckets, indexed by 𝑏=1,2,…,𝐵  
  
3\. Total Order Size:  
&nbsp;&nbsp;&nbsp;&nbsp;Q shares must be fully traded by the end of bucket B.  

4\. Fill Rates:  
At the beginning of each bucket 𝑏, you have predicted fill rates:  

$$  
F_b^A, F_b^N, F_b^P  
$$  
where  
$$  
0 ≤ F_b^A, F_b^N, F_b^P ≤ 1,
$$  

and these represent the fraction of posted shares expected to fill for Aggressive, Neutral, and Passive, respectively, in bucket b. Re-evaluation: After each bucket completes, you see the realized fills and update your predictions for the next bucket:  

$$  
(F_{b+1}^A, F_{b+1}^N, F_{b+1}^P) ← Update\ using\ realized\ fills\ in\ bucket\ b.
$$  

5\. Trajectory Constraints  
A typical VWAP/TWAP style constraint requires you to keep your cumulative executed quantity within an upper and lower envelope.
  
Let L(b) and U(b) be the fractional lower and upper bound on the cumulative fraction of Q by bucket b.   
In each bucket b, you aim for your cumulative fills to be in:  

$$  
L(b) · Q ≤ ∑(k=1 to b) [fills in bucket k] ≤ U(b) · Q
$$  

6\. Allocation Variables:  
  
$$  
x_b^A, x_b^N, x_b^P  
$$  
  
be the number of shares posted by Aggressive, Neutral, and Passive, respectively, in bucket b.  
The realized fill in bucket b for each agent is approximately:  

$$
x_b^A · F_b^A,   x_b^N · F_b^N,   x_b^P · F_b^P.
$$  

7\. Cost Function:  
Since cost rates are 2, 1, and 0 for the three agents, cost in bucket b is:  

$$  
Cost_b = 2 x_b^A + 1 x_b^N + 0 x_b^P = 2 x_b^A + x_b^N.
$$  

The overall objective is to minimize the total cost:  

$$  
Total Cost = ∑(b=1 to B) Cost_b.
$$  




  
**Main Constraints**  
1\. Complete the order by the end  

$$  
\sum_{b=1}^{B} \bigl(x_b^A F_b^A + x_b^N F_b^N + x_b^P F_b^P\bigr) = Q  
$$  

2\. Trajectory bounds  
For each bucket b:  

$$
L(b) \cdot Q \le \sum_{k=1}^{b} \bigl(x_k^A F_k^A + x_k^N F_k^N + x_k^P F_k^P\bigr) \le U(b) \cdot Q
$$

3\. Volume Capping
You can also enforce an upper limit on total posted shares each bucket (e.g., for credit or risk constraints), or require that  

$$  
x_b^A + x_b^N + x_b^P \le [max\ participation\ rate] \cdot [market\ volume] - [actual\ fill\ qty]  
$$ 

4\. Exploration (randomness)  
Each bucket 𝑏, choose allocations by a cost-minimizing rule, but alos incorporate a stochastic exploration term  
to refine fill-rate estimates.  

$$  
x_b^i = (1 - ε_b) * x_b^i(best_guess) + ε_b * x̃_b^i(random)  
$$ 

5\. Re‐estimate fill rates  
After each bucket, you re-estimate fill rates based on the actual fill ratio for each agent.  
