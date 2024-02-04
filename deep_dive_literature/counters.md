## Basics on Counters

**Guage vs Counters**

Guages:

- Go up and down
- Holds the current state of system
- Sum of values over time doesn’t make sense

Counters:

- Starts from zero
- Always goes up apart from resets
- Most common use case is counting events

**Note :** This definitions is applicable to VM compatible system, Prometheus based system behave differently for counter functions

**How to use Counters**

There are 3 functions that are used with counters to extract the meaningful information from the signal.
- increase
- rate
- irate

1. **increase**: It is used to calculate increase in signal value for a given lookbehind window.
   ```
        increase(metric[d])
   ```
   
   - If we want to calculate increase from current time at **t** till **t - d**
   - If value of signal at t - d is V<sub>t-d</sub> and value at t is V<sub>t</sub> then
     <code>
      increase => (V<sub>t</sub> - V<sub>t-d</sub>) 
     </code>
   - There are a few nuansces to this, will be covered later.
  
   <img width="1382" alt="Screenshot 2024-02-04 at 3 08 58 PM" src="https://github.com/pree-dew/promql-by-use-cases/assets/132843509/72750e05-5099-4b05-9c11-f82fb51609b2">


 2. **rate**: It is used to calculate the average per second increase rate of the signal.
     ```
         rate(metric[d])
     ```
    - If we want to calculate increase from current time at **t** till **t - d**
    - If value of signal at t - d is V<sub>t-d</sub> and value at t is V<sub>t</sub> then
      <code>
        rate => (V<sub>t</sub> - V<sub>t-d</sub>) / (t - (t - d))
      </code>
    - There are a few nuansces to this, will be covered later.

    <img width="1382" alt="Screenshot 2024-02-04 at 3 02 13 PM" src="https://github.com/pree-dew/promql-by-use-cases/assets/132843509/56045337-7b73-4640-9de4-f396cd036347">
    

3. **irate**: It is used to per second increase over only last 2 samples of a given lookbehind window.
    ```
         irate(metric[d])
     ```

---

There are nuansces when it comes to understanding how **rate** and **increase** work under the hood. 

  First **let's see how to decide the lookbehind window**, what is the impact of not selecting the right lookbehind window.
  In the screenshot at 13:32, rate didn't return any value because of lack of data points as it needs atleast 2 data points.

  In case of any lag, network issue, high scrape interval or sparse nature of metric if rate function received less number of data points
  then it will not return any result so as a precautionary measure it is advisable to take the lookbehind window = 4 times the scrape interval.

  **Compare** Query 1 with Query 2. The later, highlights what will happen if we take 4 times the scrape interval. We got the correct values even when we stopped receiving data at 13:37.

![Screenshot 2024-02-04 at 11 28 37 PM](https://github.com/pree-dew/promql-by-use-cases/assets/132843509/317c6a1b-0625-458a-a2de-e264c2e9e905)


---
**WIP**
It becomes tricky to understand the behaviour of these function because most of the time scrape interval doesn't align with the boundries of our queries. 

For example:
We want to find rate or increase from time **t** till **t - d**, where t and t - d align with minute boundary while t1, t2, t3.. are timestamps when we 
received the data. Behaviour of this output varies in different TSDBs.

```
s1    s2    s3    s4    s5    s6    s7    s8
t1    t2    t3    t4    t5    t6    t7    t8
          | <------- d-----------> |
                                   t
```


     
