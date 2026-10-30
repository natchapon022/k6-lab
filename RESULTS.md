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
| A5   | Soak         | 2     |       |       |     |        |        |

## Part B

1.
2.
3.
4.
5.
6.
7.
8.
9.