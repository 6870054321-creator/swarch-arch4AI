# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:**
chatchai panpreeda
**Student ID / รหัสนิสิต:**
6870054321
This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:** a. owner alerts

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Performance (latency) | เจ้าของร้านต้องได้รับ push notification ภายใน 2 วินาที หลังลูกค้าโพสต์รีวิวเชิงลบ |
| 2 | Capacity / Throughput | ระบบต้องรับรีวิวได้ 40 รายการต่อวินาทีในช่วงพีค ขณะที่ large model รับได้สูงสุด 20 calls ต่อวินาที |
| 3 | Accuracy (precision ของ alert) | การพลาด alert ไปบ้างยอมรับได้ แต่ alert ที่ผิด (แจ้งว่าลูกค้าไม่พอใจ ทั้งที่จริงพอใจ) ยอมรับไม่ได้ |

## 2. Design / การออกแบบ

Your topology, the patterns you add, and a container-level diagram (C4 level 2). The C4 diagram must be written in Mermaid.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

```mermaid
C4Container
    title CampusEats owner alerts (Scenario A): container diagram
    Person(customer, "Customer", "โพสต์รีวิวร้าน")
    Person(owner, "Restaurant owner", "ได้รับ push notification เมื่อมีรีวิวเชิงลบ")
    System_Boundary(campuseats, "CampusEats") {
        Container(gateway, "API gateway", "Python, FastAPI", "จุดเข้าเดียวของ client")
        Container(review, "Review service", "Python, FastAPI, small model (library)", "บันทึกรีวิว, รัน small model ในตัว (cascade), fallback เป็น pending, ส่ง alert")
        Container(inference, "Inference service", "Python, FastAPI, large model", "large model + prediction cache รับเฉพาะรีวิวที่ small model ไม่มั่นใจ")
    }
    System_Ext(push, "Push notification service", "ส่ง notification ไปยังมือถือเจ้าของร้าน")
    Rel(customer, gateway, "ส่งรีวิว", "HTTP/JSON")
    Rel(gateway, review, "ส่งต่อ request", "HTTP/JSON")
    Rel(review, inference, "POST /predict เมื่อ confidence < 0.6 (timeout 1 s)", "REST")
    Rel(review, push, "ส่ง alert เมื่อ label = negative", "HTTPS")
    Rel(push, owner, "push notification")
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| t2-0.6 | cascade threshold 0.6: accuracy 91.5% (ผ่าน ≥ 90%), large calls 38% → 40 × 0.38 = **15.2 calls/s** (< 20) และ p95 = 536 ms (< 2 s) |
| t3-cache | cascade 0.6 + cache: large calls 31% → 40 × 0.31 = **12.4 calls/s** (< 20), cache hits 36%, accuracy 91.5%, p95 = 560 ms |
| t0-large | T1 (ส่งทุกรีวิวไป large): large calls 100% → 40 × 1.00 = **40 calls/s** เกินขีดจำกัด 20 calls/s |
| t1-after | fallback: เมื่อ large model ค้าง 10% ระบบมี errors = 0 (รีวิวถูกบันทึกเป็น pending 26 รายการแทนการล้มเหลว) |


## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** ใช้ Hybrid topology (T3) ร่วมกับ cascade, cache และ fallback สำหรับ owner alerts

**Status:** Proposed

**Context:** เจ้าของร้านต้องได้รับ alert ภายใน 2 วินาทีเมื่อมีรีวิวเชิงลบ ช่วงพีคมีรีวิว 40 รายการต่อวินาที แต่ large model รับได้สูงสุด 20 calls ต่อวินาที การพลาด alert ไปบ้างยอมรับได้ แต่ alert ที่ผิดยอมรับไม่ได้

**Decision:** ใช้ T3 Hybrid โดยรัน small model เป็น library ภายใน review service และเรียก large model ผ่าน inference service เฉพาะรีวิวที่ small model มี confidence ต่ำกว่า 0.6 (cascade) เพิ่ม prediction cache หน้า large model และตั้ง timeout 1 วินาที ถ้า large model ไม่ตอบ ให้บันทึกรีวิวเป็น pending (fallback)

**Consequences:** 
- Positive: ใช้ large model เพียง 12.4 calls/s ที่ช่วงพีค (t3-cache) ต่ำกว่าขีดจำกัด 20 calls/s, p95 ประมาณ 560 ms อยู่ภายใน 2 วินาที, accuracy 91.5% และเมื่อ large model ค้าง รีวิวไม่ล้มเหลว (t1-after errors = 0)
- Negative: ----

**Alternatives considered:** at least one topology you rejected, and why.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*

> T1 (model as a service, synchronous): ไม่เลือก เพราะทุกรีวิวต้องเรียก large model ทำให้ต้องใช้ 40 calls/s ที่ช่วงพีค (t0-large) ซึ่งเกินขีดจำกัด 20 calls/s
