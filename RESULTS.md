# K6 lab results

## Part 0

- Step 0.1, http_req_duration: 275.86 ms (avg = min = med = max = p(90) = p(95) because there was only 1 request)
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 75 requests, 3.71 req/s, p(95) = 569.96 ms. The rate stays below 5 because one iteration of a virtual user is one request plus sleep(1). The average iteration took 1.34 s (about 296 ms response time + 1 s sleep + overhead), so each VU sends about 0.75 requests per second, and 5 VUs send about 3.7 requests per second.
- Step 0.6, exit code of the normal run and of the run with P95=300: 0 for the normal run (all thresholds passed), 99 for the run with P95=300 (the p(95) threshold on http_req_duration was crossed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: All 3 rules passed (p(95)<1000, p(99)<1500, http_req_failed rate<0.01), exit code 0. Measured p(95) = 377.48 ms. p(99) is not printed in the summary, but it is below 1500 ms because the maximum was 400.37 ms. After I changed p(99)<1500 to p(99)<50, the p(99) rule failed (K6 said the thresholds on http_req_duration had been crossed) and the exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it: The request used lowercase names ('pepperoni', 'knife'), but the API uses capitalized names (for example the tools are "Knife", "Pizza cutter" and "Scissors", as printed by explore.js). The API did not recognise the excluded names, so some pizzas still had pepperoni or needed a knife. In my run before the fix, 15.11% of checks failed: 'no pepperoni' failed 11 times out of 75 and 'no knife needed' failed 23 times out of 75, while 'status is 200' always passed. The pizza is chosen at random, so one manual request could easily return a correct pizza by luck, while many requests under load made the failures visible. After I used 'Pepperoni' and 'Knife', all checks passed.

## Part A results

| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     |   15  | 310.33|375.52|      0|  100%      |
| A2   | Average load | 3     |  1.32 | 374  | 774    |  0      |  100%      |
| A3   | Stress       | 5     |  2.20  | 372.5 | 479.12 | 0 |100%   |
| A4   | Spike        | 5     | 1.02  | 384.55 | 481.62| 0 |  100%  |
| A5   | Soak         | 2     | 0.99  | 375.26 | 486.8 | 0 | 100%   |

## Part B

1. Complete Part A results table as filled above.
2. Because each iteration includes think time (`sleep`), a single user takes more than 1 second per iteration, keeping the total request rate below 5 req/s.
3. Response times remained stable because the peak load (5 users) was well within the system's operational envelope.
4. Response times returned to baseline within a few seconds after the spike ended.
5. Metrics remained stable during the 10-minute run. A longer soak test could reveal memory leaks or connection pool exhaustion over time.
6. Percentiles like p(95) better represent typical user experience without being skewed by outlier response times like averages are.
7. 1. Variable network latency over public internet. 2. Fluctuating traffic from other users sharing the public site.
8. Uncontrolled high-load testing against third-party sites can act as a Denial-of-Service (DoS) attack and violates service terms.
9. 
   - Vague requirement: "The feed page should load fast."
   - Testable requirement: "95% of requests to the feed endpoint must finish within 800 ms with an error rate under 1%."
   - K6 Thresholds:
     ```javascript
     thresholds: {
       http_req_duration: ['p(95)<800'],
       http_req_failed: ['rate<0.01'],
     }
     ```