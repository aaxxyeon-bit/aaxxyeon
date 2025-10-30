<p align="center">
  <img src="https://raw.githubusercontent.com/aaxxyeon-bit/images/refs/heads/main/banner.png" alt="Banner" width="1000" height="250"/>
</p>


<h1 align="center">🧪 Comprehensive Web Application Performance Testing & Analysis</h1>

<p align="center">
  <strong>Author:</strong> Badrisha Aaina <br>
  <strong>Student ID:</strong> [Insert ID Here] <br>
  <strong>Group:</strong> [Insert Group Here] <br>
  <strong>Course:</strong> ITT440 – Web Application Performance Testing
</p>

---

## 🧩 1. Introduction  

### 🎯 Objective  
The purpose of this project is to **design, execute, and critically analyze a performance test plan** for a real-world web application using **Grafana k6**. The test aims to assess the **scalability and stability** of the target web application, identifying potential bottlenecks under varying user loads.

### 🌐 Target Web Application  
**Website:** [https://blazedemo.com](https://blazedemo.com)  
BlazeDemo is a simple flight booking demonstration site often used for testing purposes. It simulates typical web functionalities such as form submission, page navigation, and dynamic content rendering.

### 🧠 Hypothesis  
It is hypothesized that **BlazeDemo** can handle moderate user loads (up to 100 concurrent users) with stable response times. However, beyond this threshold, the application may begin to experience **performance degradation** due to limited server capacity or backend response delays.

---

## ⚙️ 2. Tool Selection Justification  

### 🧰 Tool: Grafana k6  
**k6** is a modern, developer-centric load and performance testing tool built for both **local and cloud-based execution**. It was chosen for the following reasons:

| Feature | Reason for Selection |
|----------|---------------------|
| 🧑‍💻 Scripting in JavaScript | Easy to customize and integrate in VS Code |
| ☁️ Grafana Cloud Integration | Allows real-time visualization and result tracking |
| 📊 Metrics & Thresholds | Supports response time, throughput, error rate, and custom KPIs |
| 🧱 Scalability Support | Suitable for simulating realistic load tests |

### 🔧 Supporting Tool: Grafana Cloud Dashboard  
Grafana Cloud provides visual insights into performance metrics such as request duration, VU ramp-up patterns, and pass/fail thresholds. This integration helps in better understanding system behavior over time.

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
