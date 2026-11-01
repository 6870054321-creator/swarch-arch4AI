# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** chatchai panpreeda

**Student ID / รหัสนิสิต:** 6870054321

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 499 | 612 | 93.5% | 100% |
| t0-small | 11 | 35 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> large model แม่นยำกว่ามาก (93.5% เทียบกับ 72.5% ของ small model ซึ่งต่ำกว่า QR3 ที่กำหนดไว้ 90%) แต่ต้องแลกกับความเร็ว โดย p50 ของ large อยู่ที่ 499 ms ขณะที่ small อยู่ที่เพียง 11 ms

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 28.7 | 233 | 346 | 20 | 0 | 92.8% |
| t1-after | 9.7 | 398 | 1013 | 0 | 26 | 93.1% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> column ที่บอกความจริงคือ errors ซึ่งมีถึง 20 รายการ และเวลาที่ run ใช้ 28.7 s (เทียบกับประมาณ 10 s ในกรณีปกติ) ค่า p50 เพียงอย่างเดียวจึงเชื่อถือไม่ได้

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> เพราะ request ที่ค้างยังจองทรัพยากรไว้ เช่น connection และ slot ของ large model ซึ่งมีเพียง 4 slots ทำให้ request อื่นที่ไม่ได้ค้างต้องรอคิวไปด้วย (t1-before ใช้เวลาทั้ง run 28.7 s เทียบกับประมาณ 10 s ในกรณีปกติ)

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> รีวิวที่ pending ควรถูกนำไปติด label ทีหลังเป็นชุด โดยใช้ topology T4 (Asynchronous / batch)

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> ถ้าตั้ง timeout ยาวเกินไป (10 s) ระบบจะกลับไปมีปัญหาแบบ t1-before คือ request ค้างนานและจองทรัพยากรไว้

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 7 | 536 | 91.5% | 38% |
| t2-0.7 | 0.7 | 384 | 566 | 93.0% | 53% |
| t2-0.8 | 0.8 | 435 | 589 | 93.5% | 64% |
| t2-0.9 | 0.9 | 469 | 591 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> เลือก threshold 0.6 (t2-0.6) เพราะ accuracy 91.5% ยังผ่าน QR3 (≥ 90%) และใช้ large model น้อยที่สุด (large calls 38%) ตาม QR4

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> เพราะรีวิวที่ต้องส่งต่อไป large model มีถึง 38% ซึ่งเกิน 5% ค่า p95 จึงเท่ากับ latency ของ large model (p50 = 7 ms แต่ p95 = 536 ms)

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> small model อาจมั่นใจผิด (overconfident) เช่นประโยคปฏิเสธอย่าง "not bad" ซึ่ง cascade จะเชื่อคำตอบนั้นโดยไม่ส่งต่อไปที่ large model

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 7 | 553 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> cache hits อยู่ที่ 35%

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> เมื่อเปลี่ยนโมเดล ผลที่อยู่ใน cache เดิมจะล้าสมัย ควรออกแบบให้มี version ของโมเดลอยู่ใน key หรือกำหนด TTL ให้ cache

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> ก่อนทำแลบ ผมเข้าใจว่า p50 ที่ลดลงแปลว่าระบบเร็วขึ้น แต่ใน t1-before ค่า p50 อยู่ที่ 233 ms ซึ่งต่ำกว่า t0-large (499 ms) ทั้งที่มี errors ถึง 20 รายการ จึงได้เรียนรู้ว่าการวัดระบบต้องดู errors ควบคู่กับ latency เสมอ เพราะค่า p50 เพียงอย่างเดียวเชื่อถือไม่ได้ว่าระบบทำงานดีจริง
