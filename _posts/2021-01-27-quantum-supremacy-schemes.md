---
layout: single
title: เมื่อไรที่ระบบควอนตัมจำลองได้ยาก? &#58; ดีไซน์การทดลอง quantum supremacy [draft]
date: 2021-02-28
categories:
  - Quantum
tags:
  - cs.CC
  - quant-ph
toc: true
toc_label: "Contents"
toc_sticky: true
toc_icon: "anchor"
excerpt: "ด้วยเหตุผลทั้งหลายทั้งปวง
คอมพิวเตอร์ธรรมดาจะไม่ถูกแทนที่ด้วยคอมพิวเตอร์ควอนตัมหรอก ในอนาคตเราจะมีคอมพิวเตอร์ควอนตัมไว้เพื่อช่วยแก้ปัญหาเฉพาะอย่างที่คอมพิวเตอร์ธรรมดาแก้ได้ยาก
อย่าง[ทอล์คของ Patrick Coles](https://www.twitch.tv/videos/917805834) ในงานประชุม [QHack](https://qhack.ai/) 2021 ของ Xanadu
ที่ผมเพิ่งดูพูดถึงหลักการดีไซน์ [variational quantum algorithms (VQA)](https://arxiv.org/abs/2012.09265)"
---

<!--- Dominik Hangleiter, [Sampling and the complexity of nature](https://arxiv.org/abs/2012.07905)-->

# เมื่อฟิสิกส์เจอกับวิทยาศาสตร์คอมพิวเตอร์

<!--[[*Motivation ต่อไปนี้มาจากเจตคติ (impression) ในปัจจุบันของผมต่อ quantum machine learning ซึ่งอาจจะไม่ถูกต้องตามความจริงมาก และน่าจะเปลี่ยนไปเมื่อผมรู้จัก QML อย่างถูกต้องมากขึ้น แต่ motivation ไม่มีผลต่อตัวเนื้อหาหลัก*]]-->

ด้วยเหตุผลทั้งหลายทั้งปวง
คอมพิวเตอร์ธรรมดาจะไม่ถูกแทนที่ด้วยคอมพิวเตอร์ควอนตัมหรอก ในอนาคตเราจะมีคอมพิวเตอร์ควอนตัมไว้เพื่อช่วยแก้ปัญหาเฉพาะอย่างที่คอมพิวเตอร์ธรรมดาแก้ได้ยาก
อย่าง[ทอล์คของ Patrick Coles](https://www.twitch.tv/videos/917805834) ในงานประชุม [QHack](https://qhack.ai/) 2021 ของ Xanadu ที่ผมเพิ่งดูพูดถึงหลักการดีไซน์ [variational quantum algorithms (VQA)](https://arxiv.org/abs/2012.09265) สำหรับหาพารามิเตอร์ $\theta$ ที่ optimize cost function ตัว cost function อาจจะคำนวณได้ยาก เราก็ให้คอมพิวเตอร์ควอนตัมคำนวณ cost function ที่พารามิเตอร์ค่าใดค่าหนึ่ง (กำหนดวิธี initialize พารามิเตอร์มาสักวิธี)
จากนั้นก็นำไปป้อนให้คอมพิวเตอร์ธรรมดาเพื่อ optimize ค่าพารามิเตอร์ เพื่อเอามาป้อนกลับให้คอมพิวเตอร์ควอนตัมวนลูปไปเรื่อยๆ

<script src="https://unpkg.com/mermaid@8.0.0/dist/mermaid.min.js"></script>
<center>
<div class="mermaid">
graph LR;
    qc(Quantum computer: <br/> Efficiently compute <br/> the cost function)-->cc(Classical computer: <br/> Optimize $\theta$);
    cc-->qc;
</div>
</center>

Patrick Coles บอกว่าวิธีนี้เป็นไอเดียที่ดีถ้าเรารู้ว่าการคำนวณตัว cost function ทำได้ง่ายบนคอมพิวเตอร์ควอนตัมแต่ยากบนคอมพิวเตอร์ธรรมดา
ปกติแล้ว cost function ใน VQA เป็น (linear combination ของ) ค่าคาดหมาย (expected value) ของ observable $O$

$$ \bra{\psi_{\rm in}}U\dgg(\theta) O U(\theta)\ket{\psi_{\rm in}}$$

ที่หาได้จากการสุ่มค่า $O$ จากเอาท์พุตของวงจรควอนตัม (quantum circuit) $U(\theta)$ หลายๆครั้งซึ่ง without loss of generality ถือได้ว่าเท่ากับการสุ่มตัวอย่างจากการแจกแจงความน่าจะเป็น

$$ p_{\x}(U(\theta)) = |\av{\x|U(\theta)|\psi_{\rm in}}|^2 $$

ที่จะเห็นบิตสตริง $\x \in \\{0,1\\}^k$ จากการวัด $k$ คิวบิตใน computational basis ($k$ อาจจะน้อยกว่าจำนวนคิวบิตทั้งหมด)
ดังนั้น cost function ที่ "ง่าย" หมายความว่าเรามีอัลกอริธึมที่รับ $\theta$ เป็นอินพุตแล้วสุ่มบิตสตริงเอาท์พุต $\x$ ด้วยความน่าจะเป็น $p_{\x}(U(\theta))$ ได้อย่างมีประสิทธิภาพเมื่อเทียบกับปริมาณทรัพยากรที่ใช้เช่นหน่วยความจำหรือเวลา
เราเรียกว่าอัลกอริธึมนี้ "จำลอง" (simulate) เซตของของวงจรควอนตัม $U(\theta)$
อัลกอริธึมตัวนี้บนคอมพิวเตอร์ควอนตัมก็คือตัววงจร $U(\theta)$ เอง (ซึ่งต้องประมวล (compile) เป็นเกตพื้นฐานได้อย่างมีประสิทธิภาพและวงจรไม่ลึกเกินไปคือจำนวนเกตเป็น polynomial ของจำนวนคิวบิต)
เดี๋ยวเราจะมาถกเถียงกันว่านิยามการจำลองแบบนี้สมเหตุสมผลหรือไม่
นอกจากนี้ในความเป็นจริงทั้งคอมพิวเตอร์ควอนตัมและคอมพิวเตอร์ธรรมดาที่ทำการจำลองจะต้องมี noise ซึ่งเดี๋ยวก็จะพูดถึงเช่นกัน

<!--แต่อย่างน้อยเราก็รู้ว่าการมีอยู่ของอัลกอริธึมที่มีประสิทธิภาพแปลว่ามันง่าย-->

แต่เราจะรู้ได้อย่างไรว่าวงจรควอนตัมไหนจำลองได้ยากบนคอมพิวเตอร์ธรรมดา?
การจะบอกว่าปัญหาใดปัญหาหนึ่งยากเราจะต้องตัดความเป็นไปได้ทั้งหมด (rule out) ของอัลกอริธึมที่มีประสิทธิภาพรวมถึงอัลกอริธึมที่ยังไม่เคยมีใครค้นพบด้วย
เป็นสาเหตุเดียวกับที่ทำให้ P $\neq$ NP พิสูจน์ได้ยากเพราะจะต้อง rule out ทุกอัลกอริธึมที่จะแก้ปัญหา NP-complete ได้อย่างมีประสิทธิภาพ
ในเปเปอร์ [VQA สำหรับการประมวลเกตควอนตัม](https://arxiv.org/abs/1807.00800)ของ Patrick Coles เขาพิสูจน์ความยากตรงนี้โดยการโยง cost function ของเขากับผลการคำนวณของโมเดล [One Clean Qubit](https://en.wikipedia.org/wiki/One_Clean_Qubit) ซึ่งเชื่อว่า[จำลองได้ยาก](https://arxiv.org/abs/1409.6777) แต่นี่ก็เป็นแค่การขยับคำอธิบายไปอีกขั้้นเท่านั้นเอง "เชื่อว่าจำลองได้ยาก" ในที่นี้หมายความว่าอะไร?

"ความเชื่อ" ในที่นี้คือข้อสันนิษฐานที่มีความเป็นไปได้ (plausible conjecture) ใน complexity theory ทำนอง P $\neq$ NP
ที่ทำให้เราสามารถ rule out การจำลองวงจรควอนตัมบนคอมพิวเตอร์ธรรมดาได้โดยไม่ rule out อัลกอริธึมควอนตัมไปด้วย [^1]
แต่ในความเป็นจริงคอมพิวเตอร์ควอนตัมที่ไม่ fault-tolerant ก็ต้องมี noise ทำให้อัลกอริธึมจำลองไม่ต้องแม่นยำมากก็ได้
สิ่งที่น่าทึ่งคือ[เปเปอร์ Boson Sampling](https://www.scottaaronson.com/papers/optics.pdf)
บอกว่า**เรายังมีโอกาสที่จะ rule out อัลกอริธึมที่ไม่แม่นยำนี้ได้**
Breakthrough นี้เปิดประตูสู่ความเป็นไปได้ของการทำการทดลองเพื่อพิสูจน์ quantum supremacy [^2] ที่[เคยเขียนไปในบล็อกนี้](https://ninnat.github.io/quantum/google-supremacy/)ตอนที่กูเกิ้ลประกาศผลการทดลอง Random Circuit Sampling 53 คิวบิต

<!--ด้วยแรงบันดาลใจจากทอล์คของ Patrick Coles,-->
โพสท์นี้เราจะว่าด้วยการดีไซน์การทดลอง quantum supremacy ที่ผมขอแบ่งเป็นห้าขั้นตอน

{:start="0"}
0. เลือกเซตของวงจรควอนตัม $\\{U\\}$ ที่ยังไม่มีใครรู้ว่าจำลองยังไงบนคอมพิวเตอร์ธรรมดา
1. โชว์ว่ามี $p_{\x}(U)$ ที่ประมาณค่าได้ "ยาก" (อัลกอริธึมที่ประมาณ $p_{\x}(U)$ ได้ทุกๆ instance จะแก้ทุกปัญหา #P ได้)
2. โชว์ว่า $p_{\x}(U)$ ประมาณค่าได้ "ยากโดยเฉลี่ย" (อัลกอริธึมที่ประมาณ $p_{\x}(U)$ ได้ใน instance ส่วนใหญ่จะแก้ทุกปัญหา #P ได้)
3. โชว์สิ่งที่เรียกว่า anti-concentration
4. *(ไม่จำเป็น)* เสนอวิธีเช็คว่าการทดลอง implement สิ่งที่เราต้องการให้มันทำจริงหรือไม่

<!--
ในหนึ่งปีสิ้นโลก (2020) ที่ผ่านมา ผมเข้าใจเรื่องนี้มากขึ้นจากการทำงาน quantum supremacy ใน[ระบบ driven many-body](https://scirate.com/arxiv/2002.11946)
ร่วมกับกลุ่มของ Dimitris Angelakis (สิงคโปร์)
กับเสนอการทดลองใหม่ชื่อว่า [Fermion Sampling](https://scirate.com/arxiv/2012.15825)
ซึ่งรวมเอาข้อดีของ Boson Sampling กับ Random Circuit Sampling เข้าไว้ด้วยกัน-->

<center>
<img src="/assets/img/2021/zoltan_advert.PNG" style="height: 700px;"/>
</center>


# Quantum supremacy กับกาน้ำชา

# คำนวณ vs สุ่มตัวอย่าง: อีกนิยามของการจำลอง


[^1]: Terhal and DiVincenzo, [Adaptive Quantum Computation, Constant Depth Quantum Circuits and Arthur-Merlin Games](https://arxiv.org/abs/quant-ph/0205133) (2004) และ Bremner, Jozsa and Shepherd, [Classical simulation of commuting quantum computations implies collapse of the polynomial hierarchy](https://arxiv.org/abs/1005.1407) (2010) Result ของโมเดล DQC1 ก็เข้าใจว่าเป็นแบบนี้

[^2]: ในเวทีนานาชาติมักจะใช้คำอื่นเช่น quantum advantage แทนด้วยเหตุผลว่าคำว่า supremacy มี connotation ทางลบทางประวัติศาสตร์[จากอคติเหยียดคนผิวสี white supremacy](https://www.scientificamerican.com/article/physicists-need-to-be-more-careful-with-how-they-name-things/)
