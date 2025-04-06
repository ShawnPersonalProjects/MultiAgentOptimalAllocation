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

and these represent the fraction of posted shares expected to fill for Aggressive, Neutral, and Passive, respectively, in bucket b.  
Re-evaluation: After each bucket completes, you see the realized fills and update your predictions for the next bucket:  

$$  
(F_{b+1}^A, F_{b+1}^N, F_{b+1}^P) ← Update\ using\ realized\ fills\ in\ bucket\ b.
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
