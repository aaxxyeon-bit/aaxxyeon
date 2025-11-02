<p align="center">
  <img src="https://raw.githubusercontent.com/aaxxyeon-bit/images/refs/heads/main/banner.png" alt="Banner" width="1000" height="300"/>
</p>


<h1 align="center">💻 Comprehensive Web Application Performance Testing & Analysis using Grafana k6 on BlazeDemo</h1>

<p align="center">
  <strong>AUTHOR :</strong> AINA BADRISHA BINTI ZAMRI <br>
  <strong>STUDENT ID :</strong> 2025197759 <br>
  <strong>GROUP :</strong> M3CS2554C <br>
  <strong>COURSE :</strong> ITT440 – NETWORK PROGRAMMING
</p>

---

## 🧩 1. Introduction  


### <ins>Objective</ins>  
The purpose of this project is to **design, execute, and critically analyze a performance test plan** for a real-world web application using **Grafana k6**. The test aims to evaluate the **scalability and stability** of the target web application and identifying potential bottlenecks under varying user loads.


### 🌐 <ins>Target Web Application</ins>  
**Target Website:** BlazeDemo [https://blazedemo.com](https://blazedemo.com)  
BlazeDemo is a public simple flight booking demonstration site often used for testing purposes. It simulates typical web functionalities such as form submission, page navigation, and dynamic content rendering. The site is designed for non-destructive testing and educational use, making it an ethical and legal choice for performance evaluation.


###  <ins>Hypothesis</ins>  
It is hypothesized that **BlazeDemo** can handle moderate user loads (up to 100 concurrent users) with stable response times. However, beyond this threshold, the application may begin to experience **performance degradation** due to limited server capacity or backend response delays.

---

## ⚙️ 2. Tool Selection Justification  

###  <ins>Selected Tool: Grafana k6 CLI</ins>
**k6** is a modern, developer-centric load and performance testing tool built for both local and cloud-based execution. It was chosen for the following reasons:

| No. | 🧩 **Feature** | 💡 **Reason for Selection** |
|:---:|:----------------:|:----------------------------:|
| 1 | Open-source & Developer-friendly | Freely available and highly customizable for performance testing without license cost. |
| 2 | Scriptable in JavaScript | Enables writing realistic user scenarios using a language familiar to most developers. |
| 3 | CLI & Cloud Options | Offers both local command-line and Grafana Cloud execution modes for flexibility. |
| 4 | Grafana Integration | Provides seamless visualization and monitoring of real-time metrics via Grafana dashboards. |
| 5 | Automation & CI/CD Ready | Can be easily integrated into automated pipelines for continuous performance testing. |
| 6 | Lightweight Design | Minimal resource usage, ideal for testing APIs and lightweight web applications. |

### 🔧 <ins>Supporting Tool: Grafana Cloud Dashboard</ins> 
Grafana Cloud Dashboard is used to visualize the performance testing results from k6.  
Grafana provides clear and interactive visual insights into key metrics such as request duration, virtual user (VU) ramp-up patterns, and pass/fail thresholds.  
This powerful visualization made it easier to observe system behavior, identify performance bottlenecks, and understand trends over time through detailed charts and dashboards.

###  <ins>Screenshot: Terminal Running k6 (BlazeDemo Scalability Test)</ins>

<p align="center">
  <a href="https://github.com/aaxxyeon-bit/images/blob/main/k6_terminal_run.png?raw=true" target="_blank">
    <img src="https://github.com/aaxxyeon-bit/images/blob/main/k6_terminal_run.png?raw=true" 
         alt="Terminal running Grafana k6 scalability test for BlazeDemo website" 
         width="800px"/>
  </a>
  <br/>
  <sub>Terminal running <code>k6 run blazedemo_scalability_test.js</code> — click image to view in full size.</sub>
</p>

---

## 🧪 3. Test Environment Setup  

| Component | Description |
|------------|-------------|
| **Testing Tool** | Grafana k6 (CLI & Cloud Integration) |
| **IDE** | Visual Studio Code |
| **Execution Mode** | Cloud Execution (`k6 cloud`) |
| **Virtual Users (VUs)** | Maximum 100 (within free-tier limit) |
| **Test Duration** | 18 minutes (ramp-up and ramp-down included) |
| **Metrics Monitored** | Response Time, Throughput, Error Rate, CPU, Memory |

---

## 🧬 4. Test Scenario & Script  

The test script simulates a **scalability test** where virtual users gradually increase to observe the performance stability of BlazeDemo.  
The stages include: warm-up, small load, medium load, high load, and cool-down.

```javascript
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { Trend } from 'k6/metrics';

let pageLoadTrend = new Trend('page_load_time_ms');

export let options = {
  stages: [
    { duration: '2m', target: 10 },   // warm-up
    { duration: '3m', target: 30 },   // small load
    { duration: '5m', target: 60 },   // medium load
    { duration: '5m', target: 100 },  // high load
    { duration: '3m', target: 0 }     // cool-down
  ],
  thresholds: {
    'http_req_failed': ['rate<0.05'],       
    'http_req_duration': ['p(95)<1500']     
  },
};

const BASE = 'https://blazedemo.com';

export default function () {
  group('homepage + search', function () {
    let res1 = http.get(BASE + '/');
    pageLoadTrend.add(res1.timings.duration);
    check(res1, { 'homepage status 200': (r) => r.status === 200 });
    sleep(1 + Math.random() * 2);

    let res2 = http.get(BASE + '/reserve.php?fromPort=Paris&toPort=Buenos%20Aires');
    pageLoadTrend.add(res2.timings.duration);
    check(res2, { 'reserve page 200': (r) => r.status === 200 });
    sleep(Math.random() * 3);
  });
}
