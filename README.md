*ฉบับยังไม่สมบูรณ์อยู่ในช่วงปรับปรุงเนื้อหาและเรียบเรียงรวมไปถึงอัพเดตความรู้เพิ่มเติมและแก้ไขให้ดียิ่งขึ้น : เนื้อหาอยู่ในระดับ concept และสามารถปฏิบัติตามได้แค่ฝั่ง back-end ได้บางส่วน รอการอัพเดตและเรียบเรียง ร่วมไปถึงเนื้อหาฝั่ง front-end (เนื้อหาส่วนของตัวอย่าง code อยู่ในช่วงแก้ไขยังไม่พร้อมใช้งานจริง มีเพียงแค่ระดับฝึกฝนเบื้องต้น)
เอกสารนี้เกี่ยวข้องกับการสร้างระบบความปลอดภัยและการยืนยันตัวตนโดยใช้ case study ของการออกแบบระบบ login และ register 
ว่าด้วยเรื่องระบบและความปลอดภัย Login และ Register

# Concept หลักการทำงานเบื้องต้น

# Register
-ผู้ใช้งานทำการกรอก username , password บนส่วน front-end
-ข้อมูลถูกส่งผ่าน HTTP ไปหา Back-end
-Back-end เก็บข้อมูลลงใน table ของ Database
-Back-end แสดงผลลัพธ์ยืนยันผลลัพธ์ของกระบวนการ register

# Login
-ผู้ใช้ส่ง username , password ลงบน Front-end 
-ข้อมูลถูกส่งผ่าน HTTP ไปหา Back-end
-Back-end ทำการเช็คข้อมูลว่า username , password ตรงกับที่เก็บใน Database หรือไม่
-ถ้า username,password ถูกต้อง Back-end จะทำการสร้าง token เพื่อเป็นตัวแทนการส่ง username,password 
หลังจากที่ Back-end สร้าง Token ขึ้นมาและส่งให้ทาง front-end ใช้ จนกว่า token นั้นๆจะหมดอายุขัย
token จะถูกใช้ส่งไปส่งกลับระหว่าง front-end และ Back-end ในฐานะ สิ่งที่ใช้ยืนยันตัวตนของผู้ใช้ เพื่อเลี่ยงการส่ง username,password ไปมา
โดยหลักการทำงานคือ การเทียบ ค่า token ที่ส่งมาพร้อมกับคำขอ ที่ทาง front-end ส่งมาให้กับ back-end ในการยืนยันตัวตน
สาเหตุที่ต้องใช้ token แทนการส่ง username,passwordไปมา ก็เพื่อที่จะได้ไม่ต้องส่ง username,password ลดความเสี่ยงจากการโดนแอบอ้างแล้วดักจับการใช้ข้อมูลของผู้อื่น

แต่เนื่องจากการเก็บ token บน database เมื่อมีผู้ใช้จำนวนมากใช้งาน จะเป็นการเพิ่มภาระให้ database 
บทเรียนหลังจากนี้จะเกี่ยวข้องกับการทำ token โดยที่ไม่ต้องเก็บบน database เรียกว่า jsonwebtoken (jwt)

# jsonwebtoken (jwt)

ในส่วนนี้เราจะมาพูดถึงโครงสร้างของวิธีการสร้าง jsonwebtoken(JWT) ขึ้นมา
โดยเนื้อหาเอกสารนี้จะแยกส่วนต่างๆภาพรวมก่อน  จากนั้นจะเป็นเนื้อหา อธิบายวิธีการทำงานทีละส่วนไป

## โครงสร้าง แบ่งเป็น 3 ส่วน

header.payload.signature

### header 
ด้านในจะมีประเภทของ token และ algoritm ในการเข้ารหัส (ละเนื้อหาส่วนนี้ไปก่อนได้ส่วนที่ใช้งานหลักๆอยู่ที่ 2 ส่วนที่เหลือ)

### payload 
เป็นแหล่งที่เก็บตัวแปรของชุดข้อมูลเช่น อายุขัยของ token (jwt เก็บอายุขัยของtoken ในตัว token เลยแทนการเก็บบน database) เป็นต้น

### signature
ส่วนที่ทำการป้องกันการปลอมแปลง token โดยหลักการทำงานเพื่อสร้างความแตกต่างมีขั้นตอนดังนี้
1. นำ header มาเข้ารหัส ด้วย base64 (base64 เป็นการแปรงรหัส(Encoding)ของเราให้กลายเป็น ข้อมูลชุดอักษรชุดหนึ่ง ซึ่งสามารถถอดไปถอดกลับรหัสกับข้อมูลได้)
2. นำ payload มาเข้ารหัสด้วย base64 
3. นำข้อที่ 1 และ 2 มาเขียนต่อกันโดยใส่จุด1จุดขั้นระหว่าง 1 และ 2 ==>  1.2
4. นำข้อ 3 และ supersecret Key  มา hash พร้อมกัน
ประเด็นในข้อ 4 มีสองประเด็นหลัก
#### supersecret key 
อันนี้คือ รหัสลับที่เราสามารถกำหนดเองได้ตามที่เราต้องการ  
#### hash 
คือ one-way-function โดยจะแปรงข้อมูลที่ได้รับมาให้กลายเป็น ข้อมูลตัวอักษรชุด1 โดยจะถอดกลับไม่ได้ (เรียกผลลัพธ์จากการ hash ว่า Digest)
โดยการ hash มีโอกาสที่ ข้อมูลที่ใส่ต่างกันแต่เมื่อ hash ออกมากลายเป็น Digest (ผลลัพธ์จากการ hash) ชุดเดียวกัน (คือซ้ำกัน) ได้
วิธีแก้ไข้ ณ ปัจจุบัน
hash เป็น function ที่ fix length คือ ผลลัพธ์จากการ hash จะมีความยาวตัวอักษรเท่าเดิมเสมอไม่ว่าข้อมูลที่ใส่เข้าไปจะยาวแค่ไหนก็ตาม
โดยมาตรฐานการ hash มีหลายแบบจำนวนความยาวของ Digest ก็จะต่างกันไปตามมาตรฐานต่างๆด้วย 
ซึ่ง JWT ใช้การ hash แบบ HMAC-SHA256 (ละไว้ที่เข้าใจได้ในเรื่องรายละเอียด)
สุดท้าย
นำข้อ 3 ใส่จุด และต่อท้ายด้วย 4
ผลลัพธ์คือ
header(base64).payload(base64).{[header(base64)+payload(base64)+super secret key]hash}

และนำ jwt เป็น token ยืนยันตัวตนผู้ใช้งาน

เนื้อหาต่อไปเป็นแนวติดเรื่องการรับส่งข้อมูลที่สามารถนำมาต่อยอดในการออกแบบระบบความปลอดภัยได้
---
วิธีเรียนรู้ (อาจจะเปป็นการเข้าใจผิดไปเอง)  !!! เป็นการเขียนอธิบายเป็นขั้นเป็นตอนตรรกะแบบแบ่งๆเป็นส่วนๆแล้วค่อยๆเพิ่มจะไม่โยนทุกอย่างมาอธิบายในบันทัดเดียว , มีบัคหรือข้อผิดพลาดขออภัยสามารถแค่นี้จริงๆตอนนี้
# Private Key , Public key

เรื่องที่ต้องรู้หลักๆ 

1. ผู้ใช้งานแต่ละคนจะมี key 1 ชุด คือ  Private key  และ  Public key

2. กลไลการทำงานเบื้องต้น :  
ตัวละครมี นาย A นายB นายC นายD
 สมมุติ  นาย A เขียนจดหมาย ต้องการส่งถึง นาย B แบบไม่ต้องการให้ คนอื่น นอกจาก นาย B เข้าใจเนื้อหาจดหมายฉบับนี้

ขั้นตอนกลไกเป็นดังนี้

อธิบายให้เห็นภาพในขั้นตอนนี้ (แบบเปรียบเปรยแบบรูปธรรม)

1. นาย A เขียนจดหมาย
2. นาย A เอาเนื้อหาจดหมายใส่ไปแปลงเป็นรหัสลับด้วย key (ตรงนี้ : นาย A ต้องตัดสินใจว่าจะใช้ Key ไหนในการแปลงระหว่าง Private หรือ Public) ก่อนค่อยเอาไปให้ นายB

อธิบายแบบสมมุติ : (ถ้าจำกติกาไม่ได้อ่านผ่านคร่าวๆไปก่อนน้อจนถึงตัวอย่าง code แล้วมาทวนกติกาดู)

 นาย A เขียนจดหมาย(เป็นกระดาษ) แล้วโยนใส่ไว้ในกล่องล็อคด้วย 'แม่กุญแจ(Key)'  
---
## กติกา
(ถ้าจำกติกาไม่ได้ช่างมันก่อนได้ไปดูตัวอย่างแล้วค่อยกลับเลื่อนมาเทียบดู) 

### ข้อที่1

 ถ้าเอา Private  มาทำเป็นแม่กุญแจ ต้องเอา Public ชื่อเดียวกันมาทำเป็นลูกกุญแจใช้ในการเปิด 

*ข้อนี้พิเศษหน่อย : 

สามารถนำ Public ของคนอื่นมาทำเป็น แม่กุญแจได้ เช่น นาย C เอา Public A มาทำเป็นแม่กุญแจ
แต่
การเอา Private มาทำเป็น แม่กุญแจ สงวนสิทธิ์ให้เจ้าของ Private ทำเท่านั้น เช่น นาย B สามารถทำ Private B เป็นแม่กุญแจ คนเดียวเท่านั้น ซึ่งในทางกลับกัน นาย B ไม่สามารถทำ Private A,C,D มาเป็นแม่กุญแจได้


ในทางกลับกัน

ถ้าเอา Public เป็นแม่กุญแจ ต้องเอา  Private ชื่อเดียวกันมาทำเป็นลูกกุญแจใช้ในการเปิด

### ข้อที่ 2

ไม่ว่าจะเอา  Public จะถูกทำเป็น แม่กุญแจ หรือ ลูกกุญแจ

ต้องเอา ลูกกุญแจ  Public หรือ แม่กุญแจ Public ไปแจกทุกคน


### ข้อ 3

ส่วน Private จะเป็น ของใครของมัน เท่านั้น ห้ามแจกให้คนอื่น


### ข้อ 4 (ข้ออธิบายนี้สำคัญเพราะเกี่ยวกับความต่างของตรรกะได้)

ถ้าเอา Private มาทำเป็นแม่กุญแจ แปลว่า  Public กลายเป็น ลูกกุญแจ แปลว่า!! : " ทุกคน" จะเป็นผู้ถือครองลูกกุญแจ Public และ มีสิทธิ์ สามารถเปิดกล่องที่มี Private เป็นแม่กุญแจได้ !! ( ) 


### ข้อ5 (กติกาข้อนี้ตั้งไว้แก้บัคการยกตัวอย่างเฉยๆ)

ทุกครั้งที่มีการส่งของ จะมีบิลส่งของเหลืออยู่ โดยในบิลส่งของจะไม่มีชื่อผู้รับหรือผู้ส่งเขียนในบิลแต่บอกแค่ว่า มีของมาส่ง และของที่ว่าตั้งอยู่ตรงไหน


(ถ้าจำกติกาไม่ได้ช่างมันก่อนได้นะมาดูตัวอย่างก่อนคร่าวๆค่อยย้อนขึ้นไปดู)

ตัวอย่าง
ที่เรียนในห้องเรียน 

-Private A(Public B)
และ
-Public B (Private A)


-Private A(Public B)
ตัวอย่างข้อนี้ เหมือนกับ การที่ นาย A  เขียนจดหมายแล้ว
1. เอาจดหมาย ใส่กล่องที่มี แม่กุญแจPublic B
แล้ว!! 
2. ยังเอากล่องที่ได้จากข้อ 1 ไปใส่กล่องที่ใหญ่กว่าที่ ล็อคด้วย แม่กุญแจ Private A
แปลว่า!!
นาย B ที่คิดจะเปิดดูจดหมาย ต้องแกะกล่อง นอกสุดก่อน (Private A)
แล้วค่อยแกะกล่องข้างใน( PublicB) อีกทีเพื่อที่จะเจอจดหมาย
(แกะจากนอกเข้าไปข้างใน)

สิ่งที่นาย B ต้องทำเมื่อได้กล่องมาจากนาย A

1. นายB ต้องใช้  ลูกกุญแจแบบ Public A (ได้มาตามกติกาข้อ2) ซึ่ง แปลว่าตอนนี้คนอื่นๆก็ถือกุญแจ Public Aเช่นกัน (กติกาข้อ4)   
เพื่อ เปิดกล่องนอก
แล้วจากนั้น
2. นาย B จะพบกล่องที่มี แม่กุญแจ Public B 
ซึ่ง
 นาย B ถือครอง ลูกกุญแจ Private B อยู่แล้วทำให้เปิดกล่องออกมาเจอจดหมายข้างในได้
แต่!!
สมมุติ นาย A วางกล่องไว้หน้าบ้านนาย B คราวนี้นาย C บังเอิญไปเห็น บิลรับของ(ที่ไม่มีผู้ส่งและผู้รับแต่แจ้งว่ามีการส่งของ)  ทำให้ นาย C  ยังไม่รู้ว่าใครเป็นคนส่งหรือใครเป็นคนรับ และบังเอิญ ไปเห็นกล่องที่ นายA ทำไว้วางอยู่พอดี
ซึ่งตอนแรกสุดนาย C จะไม่มีทางรู้ได้เลยว่าใครเป็นคนส่งและใครเป็นคนรับ แต่เห็นแค่กล่อง พอนาย C เดินไปดูที่กล่องจะพบว่า
มี แม่กุญแจ Private A ล็อคกล่องอยู่ ซึ่งนาย C ก็จะได้รับ ลูกกุญแจ Public A (ตามกติกาข้อที่2) เช่นกันกับคนอื่นๆ 
ทำให้
นาย C เปิดกล่องนั้นได้ และจะพบกับกล่องที่มี แม่กุญแจ Public B 
ถึงแม้ว่า
นาย C จะเปิดกล่องในสุดเพื่อดูจดหมายไม่ได้ แต่นาย C จะสามารถ
 สันนิฐานได้ว่า 
นาย A ส่งจดหมาย ถึงนาย B แน่นอน เพราะ
1. แม่กุญแจทำจาก Private ซึ่งสิทธิ์การทำ แม่กุญแจแบบ Private สงวนให้เจ้าชอง Privateทำเท่านั้น (ตามกติกาข้อ2)
2. กล่องข้างในสุดต้องใช้  ลูกกุญแจ Private B เปิด ซึ่งผู้ครอบครองเป็น นายB (ตามกติกาข้อ 3)

ต้วอย่างต่อมา
Public B (Private A)
คราวนี้เป็น นาย A ส่งจดหมายถึง B เหมือนเดิม
โดยที่ 
1. นาย A เอาจดหมายใส่กล่องที่มี แม่กุญแจ Private A ล็อคไว้
แล้ว
2. เอากล่องที่ได้จากข้อ 1 ไปใส่กล่องที่ใหญ่กว่าอีกชั้นแล้วล็อคด้วยแม่กุญแจ Public B
สิ่งที่จะเกิดขึ้น

ถ้าคนอื่น นอกจาก นาย B มาเห็นกล่องนี้จะเปิดไม่ได้เพราะ 
ต้องใช้ ลูกกุญแจ Private B ในการเปิดกล่องนอกสุด
สมมุตินาย B เปิดกล่องนี้สำเร็จ และพบว่า อ้าว เจอกล่องที่ แม่กุญแจ เป็น Private A 
ซึ่ง แปลว่า นาย A เอา Private มาทำแม่กุญแจ ทุกคนจะได้ ลูกกุญแจ Public A มาอยู่แล้ว แต่มีแค่นาย B เท่านั้นที่เปิดกล่องชั้นนอกสุด(Public B)ได้ในตอนแรก


ประเด็นที่น่าสนใจ

1. แปลว่า เราทำให้มันสับซ้อนขึ้นได้ด้วยการใช้  private,public คนละ 2 ชุดแทนที่จะเป็นชุดเดียว 
2. แบบ private A(public B) ในอีกแง่ใช้ในฐานะ ลิขสิทธิ์ ได้เลยเพราะทุกคนรู้หมดว่าใครส่ง
 


---

เนื้อหาต่อไป การป้องกันการถอดรหัสให้มีประสิทธิภาพมากขึ้น

จากเนื้อหาก่อนหน้านี้ในเรื่อง วิธีการ hash 
ยังมีจุดอ่อนคือ
1. การที่ฝ่าย hacker สามารถสร้าง library hash ด้วยการไล่ตัวอักษรแล้วนำไป hash เก็บเป็น Library 
เพื่อนำ Library นั้นๆไปเทียบเรียกวิธีนี้ว่า Rainbow table
2. hacker สามารถ ไล่ตัวอักษรแล้วนำไป hash เรื่อยๆเพื่อวัตถุประสงค์ให้ผล hash นั้นตรงกัน (ไม่สนใจตัว  username,password แค่หวังให้ hash ตรงกันก็พอ เนื่องจาก hash เป็น function fix length โอกาสซ้ำจึงเกิดขึ้นได้) เรียกวิธีนี้ว่า brute-force
วิธีแก้ไขจุดอ่อนนี้จะเรียกว่า bcrypt 
คือการนำข้อมูลที่เราต้องการจะ hash มาเพิ่ม ชุดข้อมูลอื่นไปด้วย ก่อนค่อยทำการ hash 
ชุดข้อมูลที่เพิ่มเข้ามาจะเรียกว่า salt จะทำให้ผลการ hash ของข้อมูลเปลี่ยนไปเรื่อยๆเพราะ salt ทำให้ฝ่าย hacker ที่ต้องการจะดักข้อมูลด้วยวิธีการข้างต้นไม่สามารถเอาข้อมูลทั้งหมดได้แต่จะได้ทีละข้อมูลต่อการ 1ช้อมูลซึ่งเสียเวลาในระดับที่เป็นไปไม่ได้ จึงถือว่าเป็นวิธีการที่ได้ผลในปัจจุบัน


---

# ภาคตัวอย่างปฏิบัติ และ ตัวอย่าง code ระบบ login และ register (อยู่ช่วงปรับปรุงเนื้อหา)


# เนื้อหาฝั่ง back-end
## tutorail basic
พื้นฐานการเขียน server (แบบยังไม่ได้เชื่อมกับ database และ sql เพื่อให้เข้าใจ flow ของการไหลข้อมูลเบื้องต้น

เนื้อหาฉบับฝึกฝน ยังไม่มีการเชื่อมกับ Database เนื้อหาอธิบายหลายๆส่วนยังไม่ครบแต่ถ้าในระดับพอลงมือทำถือว่าครบท้วนระดับหนึ่งแล้ว
file จัดการเกี่ยวกับ server ข้างในจะประกอบไปด้วยส่วนหลักๆคือ 
1.file : controller
2.file : routes
3.file : index

controller : ไว้รวบรวม function ให้ตัว web เรียกใช้งาน
routes : จัดการ url path เพื่อชี้ทาง,กำหนด path ให้ web ไปที่ controller ว่าจะใช้ function ไหนใน controller
index : จัดการ port , import ตัว express มาจาก node เพื่อใช้งาน ,    จัดการ middle ware ,จัดการการทำงานผ่าน routes (เฉพาะงานนี้) *(รออัพเดตเนื้อหา)

****
ถ้าจะรัน server คำสั่งคือ nodemon เว้นวรรค (ชื่อไฟล์) ตัวอย่าง nodemon index (ถ้าคำสั่ง error อาจจะเป้นเพราะไฟล์นั้นไม่ได้ติดตั้ง node ไว้ลอง npm i ก่อน nodemon (ชื่อไฟล์)) 


วางโครงสร้าง backend
**ข้อควรระวังเล็กๆน้อยๆเนื้อหาเอกสารนี้ใช้คำว่า folder กับ file แยกกันอย่างชัดเจนจะไม่มีการใช้คำว่า ไฟล์ แทน folder (หรือบังเอิญ human error ขออภัย เจตนาหลักคือ แยก 2 คำนี้อย่างชัดเจนเป็นหลัก)
-สร้าง folder สำหรับเก็บ backend
-เปิด terminal 
[*ข้อควรระวัง : ถ้าคำสั่งใน terminal เกิด error ให้เช็ค path ของ terminal ว่าตรงกับไฟล์ Backend หรือไม่ อาจจะเกิดจากการที่ terminal กำลังชี้ไปที่ fileลูกในfolderของ Backend แทนที่จะชี้ไปที่ folder Backend เช่น controller,routes เป็นต้น
 ถ้าpath ไม่ตรง  วิธีแก้ไขที่เป็นไปได้ 
กรณีที่ 1.ถ้าเราใช้ vscode เปิดfolder Backend ตรงๆให้คลิกพื้นที่เปล่าตรงที่เก็บไฟล์ด้านซ้ายมือของหน้าต่าง Vscode (หมายถึงหน้าต่างที่แสดงข้อมูลว่า vscode กำลังเปิดใช้งาน folderใด )  ให้มีกรอบสีฟ้าขึ้นรอบๆ พื้นที่ background พื้นที่ส่วนนั้น แล้ว คลิกขวา เลือก open terminal(คำสั่งเขียนยาวๆหน่อยแต่จะประมาณนี้) path terminal จะถูกเปลี่ยน  )
กรณีที่ 2.ถ้าเราใช้ vscode โดยเป็นfolderรวมหลายๆอย่างไว้ และ Backend เป็น 1 ในfolderลูกของ folderที่ใช้เปิด มองไปด้านซ้ายมือ ให้คลิกขวาที่ชื่อของFolder Backendของเรา   จากนั้น เลือก open terminal (คำสั่งยาวๆแต่ประมาณนี้)  path terminal จะถูกเปลี่ยน)
]
-npm i init -y
-npm i express (*ข้อควรระวัง : command ถ้า terminal ทวงว่าไม่มี command นี้คือไฟล์นี้ยังไม่ได้ลง node ดังนั้นต้องติดตั้ง node ก่อน : ใส่คำสั่ง npm i ใน terminalแล้วค่อย npm i express)
-สร้างไฟล์ index.js
ลำดับการเขียนเนื้อหาใน ไฟล์ index 

-const express = require('express')
-const app = express()


const todoRouter = require('./routes/todo')
(ความหมาย : การกำหนด url path ให้app ทำงานผ่าน url path )


เนื้อหา middle ware

-app.use(express.json())
-app.use(express.urlencoded({extended:false}))
(ความหมาย : การจัดการในการถอดรหัสข้อมูล)

เนื้อการเชื่อมต่อ routes
app.use('/todo',todoRouter)
(ความหมาย : คือการสั่งให้การจัดการ function ของเว็ปไซต์ในไฟล์ routes(พวก get, put ,post,patch,delect))
(เนื้อหาอ้างอิงจากไฟล์ routes)

เนื้อหาของ port
-app.listen(8000,()=>{
console.log('Server is running on port 8000')
}
)


ตัวอย่างเนื้อหาในไฟล์ index.js

const express = require('express')
const app = express()

const todoRouter = require('./routes/todo')

app.use(express.json())
app.use(express.urlencoded({extended:false}))

app.use('/todo',todoRouter)

app.listen(8000,()=>{
console.log('Server is running on port 8000')
})


วิธีเขียนไฟล์ routes
สร้าง folder routes เก็บไฟล์ (ชื่แไฟล์routesของโปรเจคนั้นๆ).js(เช่น todo.jsเป็นต้น) เพื่อเก็บ url path เพื่อนำทาง app ไปหา function ในไฟล์ controller(controller ไว้เก็บ function ของ app)

ขั้นที่1
const express = require('express')
(ความหมายให้ใช้ดึง express มาใช้งานจาก node)
const router = express.Router()


ขั้นที่2 : กำหนดให้ routes ดึง function จากไฟล์ตาม path
const todoControllers = require('../controllers/todo')
(ความหมายตัวแปร todoControllers ให้ถือเป็นการดึงข้อมูล จากข้างใน folder ชื่อ controllers ซึ่งดึง file ชื่อ todo (แปลว่า folder controllerจะมีfileชื่อ todo อยู่) )

ขั้นที่3 : กำหนด วิธีเขียน path url 
router.post('/add',todoControllers.addTodo)
(ความหมาย: เวลาจะเรียกใช้งาน function ต้องเขียน url แบบนี้ : (url เช่น localhost:8000)/add  คือการเรียก function addTodo จากไฟล์ todoController)
router.get()

router.patch()

router.delete()

module.exports = router
(express ยังไม่รองรับ ES6 ดังนั้นจึงใช้การ export ไฟล์แบบ ES5 : บันทัดนี้คือ export นั้นเอง)



เนื้อหาใน routes


const express = require('express');
const router = express.Router();
const todoControllers = require('../controllers/todo');

router.post('/add',todoControllers.addTodo)

router.get('/',todoControllers.getAllTodo)

router.get('/:id',todoControllers.getTodoById)

router.delete('/:id',todoControllers.removeTodo)

router.patch('/:id',todoControllers.updateTodo)

module.exports = router


เพิ่มเต็ม
flow ของข้อมูลในหน้านี้ (รออัพเดตเนื้อหา)

-router เป็นตัวแปรที่ใช้งาน express (ผ่านตัวแปร express บันทัดด้านบน)
-เนื้อหาด้านในเป็นการบอกว่า router แต่ละ path จะไปเรียกใช้งาน function จากไฟล์ไหน
-ประกาศส่งไฟล์ router ออกไปใช้งาน 


***กรณีที่ยังไม่เชื่อม database เนื้อหาสำหรับฝึกฝนควบคู่ไปกับ Postman
สร้างfolder controllers  ไว้เก็บ file constroller ของproject 

let todoList = [] *(ใช้แทน database ไปก่อน)


ขั้นที่1
-let id = 1 *(เอาไว้เพิ่ม  id ใน function create (ในที่นี้คือ function addTodo)) 
ส่วนสำคัญ
ขั้นที่2
สร้างตัวแปรเพื่อเก็บ function  เช่น

const addTodo = (req,res)=>{
const {task} = req.body
const newData = {
id:id++,
task
}
todoList.push(newData)
res.status(201).send(newData)
}

(ควาามหมาย :
 ประกาศตัวแปร addTodo 
(req,res): เมื่อมีการ request (เรียกใช้ตัวแปร addTodo) จะทำการ response (ส่งคืนฟัง์ชั่นข้างหลังทีเขียนต่อจากนี้)

req เป็น params ไว้ใช้ใน function

(พื้นที่เขียน function)

res.status(201) = แสดงสเตตัลจากการส่งฟังค์ชั่นไปใช้งานใน App
.read(...) = จะให้แสดงส่วนไหนเมื่อทำการกด send ให้ใส่ไปใน ...  (เหมือนกับ console.log(...) นั้นเอง)
)

**สำหรับฝึกฝนในกรณีที่ยังไม่เชื่อมกับ Database
[*ข้อควรระวังในหัวข้อ function ประเภท addหรือ create  req.body และ req.query ไม่เหมือนกัน
 ถ้าเขียนฟังค์ชั่นให้รับค่าจาก req.body อย่างตัวอย่างข้างต้นในกรอกค่า value ในหัวข้อ body ,
 ถ้าเขียนฟังค์ชั่นที่รับค่าจาก req.query ให้ใส่ค่า value จาก params ได้เลย]

const  getTodoById = (req,res)=>{

}

const getAllTodo = (req,res)=>{

}

const addTodo = (req,res)=>{

res.status(201).send()
}

const updateTodo = (req,res)=>{

}

const removeTodo = (req,res)=>{

}


(*ส่วนนี้สำคัญ : คือการส่งออก function อะไรบ้างจาก file นี้ (ถ้าเขียน function ด้านบนแล้วไม่เอามาใส่ที่ส้วน export ก็จะไม่มี function นั้นๆออกไปจาก file นี้นะ))
module.exports = {
    getAllTodo,
    getTodoById,
    addTodo,
    updateTodo,
    removeTodo,
}

---
ตัวอย่างเนื้อหาในไฟล์ controllers (ยังไม่ได้เชื่อมกับ Database)

let todoList = []
let id = 1

const addTodo = (req, res) => {
    let { task } = req.body
    let newList = {
        id: id++,
        task,
    };
    todoList.push(newList);
    res.status(201).send(newList)
}

const getAllTodo = (req, res) => {
    res.status(200).send(todoList)
}


const getTodoById = (req, res) => {
    const targetId = Number(req.params.id);
    const targetList = todoList.filter(listItem => listItem.id == targetId)
    res.status(200).send(targetList)
}

const removeTodo = (req, res) => {
    const targetId = Number(req.params.id);
    const targetList = todoList.filter(listItem => listItem.id != targetId)
    todoList = targetList
    res.status(204).send(todoList)
}

const updateTodo = (req,res)=>{
    const targetId = Number(req.params.id)
    const targetList = todoList.findIndex(listItem=>listItem.id==targetId)
    const { task } = req.body
    const updateList = {
        id:targetId,
        task
    }
   todoList[targetList] = updateList;
   res.status(200).send({message :`Update data in id: ${targetId} already `})
}


module.exports = {
    addTodo,
    getAllTodo,
    getTodoById,
    removeTodo,
    updateTodo,
}

---


## Tutorial 
server ฉบับเชื่อมกับ database sql (อัพเดตเนื้อหายังไม่ครบถ้วน) *เนื้อหาเอกสารนี้ยังไม่เสร็จ

folder
./tutorial/authentication back-end

Tutorail

npm i init -y express sequelize mysql2 cors express bcryptjs passport passport-jwt jsonwebtoken dotenv

touch .gitignore
(สร้างไฟล์ gitinore ขึ้นมาเพื่อจะบอกว่าเรจะะไม่อัพไฟล์ขึ้นgit)

สร้างไฟล์ .env

เนื้อหา .env : อธิบาย : เป็นไฟล์ที่เรียกใช้งานได้ทั้ง global โดยการเรียกจะต้องเรียก ก่อนที่จะใช้งาน ทรัพยากรของ .env
(SECRET_KEY เป็นตัวใหญ่เพราะเป็น const (ตัวแปรที่เปลี่ยนค่าไม่ได้))
[
port =8000
SECRET_KEY = C0|)3C4/\/\p 
]

สร้างไฟล์ index.js

เนื้อหา

require('dotenv').config() 
(ความหมายคือการเรียกใช้ไฟล์ .env โดยที่ไฟล์ env เป็นไฟล์ประเภท config) - (ต้องเพิ่มความเข้าใจคำว่า config)

[เกร็ดความรู้ (ต้องเพิ่มควาเข้าใจ)
*ถ้า Run javascript บน Node , process จะกลายเป็น global object]

const express = require('express')
(ประกาศตัวแปลให้ express เป็นการดึง express มาใช้งานจากข้างใน Node)

const cors = require('cors')
(ประกาศ cors เรียก cors มาใช้  คร่าวๆเกี่ยวข้องกับความปลอดภัย (เพิ่มเติมความเข้าใจหัวข้อความปลอดภัย CORS))

const app = express() 
(ประกาศให้ app เป็นการใช้งาน  express)

เนื้อหา middle ware 
app.use(cors())
app.use(express.json())
app.use(urlencoded({extened:false}))


เนื้อหาของการเชื่อม PORT
app.listen(process.env.PORT,()=>{
console.log('Server is running')
})
[ความหมาย : คือ การเชื่อม port โดยดึงจาก process ของ node โดยไปหาจากไฟล์ .env]

ตัวอย่าเนื้อไฟล์ index

require('dotenv').config();
const express = require('express');
const cors = require('cors');
const { urlencoded } = require('express');

const app =express()

app.use(cors());
app.use(express.json())
app.use(urlencoded({extended:false}))

app.listen(process.env.PORT,()=>{
console.log('Server is running')
})

---


*อธิบาย : ขั้นตอนนี้คือการใช้ภาษา sql ในการจัดการ table ใน database โดยเราจะทำการจัดการด้วยภาษา Javascript แทนการใช้ภาษา sql

ทำการเปิด terminal แล้วพิมพ์คำสั่งตามลำดับ 

1. npm install mysql2 sequelize
2. npm install mysql sequelize-cli -g
3. sequelize-cli init:config (เป็นการสร้าง config)
4. แก้ไข user, password, Database ใน config/config.json
5. สร้าง Database -> sequelize-cli db:create
6. สร้างfolder model -> sequelize-cli init:models

สร้างไฟล์ (ชื่อที่จะใช้ใน project นี้).js (เช่น  TodoList.js) ใน Folder models

module.export = (sequelize,DataType) => {
const TodoList = sequelize.define("Todolist",{
Task:{
	unique: true,
	allowNull: false,
	type: DataType.STRING
}
},{
	tableName: 'todolists',
	timestamps: fasle

})

return TodoList;
}


---

สร้างไฟล์ User

module.exports = (sequelize, DataType) =>{
const TodoList = sequelize.define("TodoList",{
    task: {
        unique :true,
        allowNull: false,
        type: DataType.STRING
    }
},{
    tableName: 'todolists',
    timestamps: false
})

return TodoList;
}
---

เปิดไฟล์ index 
เพิ่มเนื้อหาเพื่ออให้ททำงานเกี่ยวข้องกับการเชื่อมไฟล์ module 
เทียบกับไฟล์ขั้นตอนที่ 1.index
---

บันทัดที่ 4 
const db = require('./models')

บัดทัดที่ 12
db.sequelize.synce().then((>{
onsole.log('ข้อความแสดงเมื่อเชื่อมต่อกับ database สำเร็จ')
})

---
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const db = require('./models')

const app = express()

app.use(cors());
app.use(express.json())
app.use(express.urlencoded({ extended: false }))

db.sequelize.sync().then(()=>{
    console.log('Sync with database complete')
})



app.listen(process.env.PORT, () => {
    console.log('Server is running')
})
---
เนื้อหาที่ควรเป็น

require('dotenv').config();
const express = require('express');
const cors = require('cors');
const db = require('./models')

const app = express()

app.use(cors());
app.use(express.json())
app.use(express.urlencoded({ extended: false }))

db.sequelize.sync().then(()=>{
    app.listen(process.env.PORT, () => {
    console.log('Server is running')
})
})




---


const login = (req,res) => {

} 

cconst register =(req,res)=>{


}








module.exports = {
login,
register,
} 
---

สร้าง Folder routes สร้าง ไฟล์ เพื่อเก็บเนื้อหาของ routes เนื้อหา routes คือ การจัดการ path เพื่อบอกว่า web จะดึง function มาจากไหน
const express = require('express')
const router = express.Router();
const { login, register} = require('../controllers/user')





router.post('/login', login)
router.post('/register',register)

module.exports =router
---
เข้าไปที่ไฟล์ข้างใน module

---
User.associate = models => {
        User.hasMany(models.TodoList, { foreignKey: 'user_id' })
    }

-ความหมาย : ชื่อไฟล์ (User) . มีความสัมพันธ์ระหว่างตาราง = ในไฟล์ models =>ดังนี้ {
	ชื่อไฟล์ . (รูปแบบความสัมพันธ์ศึกษาเกี่ยวกับ relation) , {เป็น foreinKey:'Entity'}
}
---
เทียบเนื้หา user ใน module
บันทัดที่ 17
module.exports = (sequelize, DataTypes) => {
    const User = sequelize.define("User", {
        name: DataTypes.STRING,
        userName: {
            unique: true,
            allowNull: false,
            type: DataTypes.STRING
        },
        password: {
            allowNull: false,
            type: DataTypes.STRING
        }
    }, {
        tableName: 'users'
    });

    User.associate = models => {
        User.hasMany(models.TodoList, { foreignKey: 'user_id' })
    }

    return User
}
---
ไฟล์ index (ด้านนอก)   *ไม่ใช่ index ใน moodule
บันทัด 14 
db.sequelize.sync({force:false})
เพื่อ!!
ทำให้ เวลาที่เราสร้างตารางจะทำให้เกิดการ  drop (ลบ) ตาราใหม่ เพื่อนำข้อมูลที่แก้แล้วมาแทน 
ถ้าไม่ force จะทำให้อัพเดตไม่ได้
---
สร้าง function register

const db = require("../models");

const login = (req,res)=>{

};

const register = async(req,res)=>{
    const { username, password,name} = req.body;
ความหมาย : req.body.user , req.body.password , req.body.name 

    const targetUser = await db.User.findOne({where: {username}})
ความหมาย : ประกาศ targetUser  ให้ทำงานแบบ await ดึงจาก database . ใช้คำสั่งในการเลือก findOne({where : (เงื่อนไข)})   findOne คือเอาอันเดียว(อันแรก)

เนื้อหา function
หน้าที่หลักๆคือการ : ให้การ register ที่มี user ซ้ำ
    if(targetUser){
        res.status(400).send({message : 'User Already Taken'})
    } else {
        const salt = bc.genSaltSync(Number(process.env.SALT_ROUND))
        const hashedPW = bc.hashSync(password,salt)

	await db.User.create({
	password: hashedPw,
	username,
	name,
})

	res.status(201)send({message:'Register complete'})
    }
	
};

module.exports = {
    login,
    register,
}
---
*รอการอัพเดตและแก้ไข้ p.s. เอกสารเนื้อหาในฝั่งของ tutorial ส่วนนี้ยังไม่เสร็จ
