# पहिला Phone Number आणि पहिला Call

> **Course:** Amazon Connect Complete Training Course  
> **Udemy शीर्षक:** Amazon Connect User Interface  
> **या video मध्ये प्रत्यक्ष:** number claim, test call, **CCP**, **After Call Work**, login  
> **UI ची सविस्तर सफर:** पुढचा lecture

<span style="color:#0b6e4f"><b>एक वाक्य:</b></span> Instance ला **phone number** द्या, त्यावर call करून **CCP** तपासा, आणि रोजच्या कामासाठी **emergency (root) login** वापरू नका.

---

## 1. Number claim

Instance तयार झाल्यावर **Get started** → **Let's go** → number चे पान.

Browser विचारेल तर दोन्ही **Allow**:

| परवानगी | का |
|---|---|
| Notifications | call आल्याची सूचना |
| Microphone | बोलता आणि ऐकता येईल |

मग दोन निवडी:

1. **Country** — number कोणत्या देशाचा  
2. **Type** — **DID** किंवा **Toll-free**

Video: **New Zealand** + **DID** → यादीतून पहिला number → **Next**.

<div style="background:#fff7ed;padding:10px 14px;border-left:4px solid #ea580c;border-radius:6px">

Free tier चा मोफत <b>DID</b> instance च्या country चा असतो. Instance <b>Sydney</b> असेल तर मोफत number <b>Australia</b> चा. Video मध्ये New Zealand निवडला — तो country वेगळा असल्याने number चा charge लागू शकतो (<code>cost.md</code>). Course जसा आहे तसा करा, bill मात्र समजून घ्या.

</div>

---

## 2. Test call आणि CCP

त्या number वर स्वतःच्या फोनवरून call करा. **Default IVR** ऐकू येईल.

**IVR** = फोनवरची आवाजाची मेनू. “1 दाबा, 2 दाबा.”

ही मेनू नमुना आहे. तुमची अंतिम मेनू नाही. ती हे दाखवते:

- **1** → agent च्या **queue** मध्ये जा  
- Queue ची **priority** बदलता येते (पुढे किंवा मागे)  
- रांग 5 मिनिटांपेक्षा कमी असेल तर: रांगेत राहा, किंवा **callback** number द्या  

Agent कडे call आला की तो छोट्या फोनवर दिसेल. हे **Contact Control Panel (CCP)** आहे. Agent चा फोन = हा screen.

| CCP वर | अर्थ |
|---|---|
| Accept | call उचला |
| Hold / Mute | थांबवा / आवाज बंद |
| End | call संपवा |
| Transfer | दुसऱ्या number वर द्या |

Transfer साठी **Quick Connects**. जतन केलेला number तिथे दिसतो. नसेल तर number टाइप करा, **country code** निवडा, call करा. यादीत थोडेच देश दिसतात. कारण खाली.

---

## 3. After Call Work

Call संपला की status आपोआप **After Call Work (ACW)** होतो.

<div style="background:#eff6ff;padding:10px 14px;border-left:4px solid #2563eb;border-radius:6px">

ACW मध्ये तुम्हाला <b>नवीन call येत नाही</b>, जोपर्यंत <b>Close contact</b> दाबत नाही.

</div>

हे जवळजवळ प्रत्येक call center मध्ये असते. Call संपल्यावर agent लिहितो, request पूर्ण करतो. त्या वेळी नवीन call **उपलब्ध** agent कडे जातो — जो ACW मध्ये नाही.

तयार असाल तर **Close contact**. मग पुन्हा calls येतात.

<span style="color:#b45309"><b>चूक:</b></span> Close contact विसरलात तर तुम्ही “व्यस्त” दिसता आणि call येत नाही.

---

## 4. Outbound — थोडेच देश का?

Instance कुठे तयार झाला, त्यावर अवलंबून Connect मुळात काही देशांनाच call करू देतो.

पहा: [Connect Administrator Guide → Service quotas](https://docs.aws.amazon.com/connect/latest/adminguide/amazon-connect-service-limits.html) → **countries you can call by default**.

देश वाढवायचे किंवा काढायचे असतील तर **service quota** बदलाची request पाठवा. ही यादी bug नाही.

---

## 5. Login — दोन दरवाजे

**Continue** केल्यावर Connect **dashboard**.

| दरवाजा | कधी | सुरक्षित? |
|---|---|---|
| Instance चा **access URL** | सामान्य login. User हवे | होय, पण अजून user नाही |
| AWS console मधला instance link → **Login for emergency access** | फक्त अडचण | <span style="color:#b91c1c"><b>नाही</b></span> |

मागच्या video मध्ये administrator तयार केला नव्हता. त्यामुळे access URL वर जाण्यासारखा user नाही.

Emergency access **AWS root account** ने Connect उघडतो. चेतावणी स्वतः सांगते: रोजच्या कामासाठी वापरू नका.

आता course साठी emergency login. नंतर वेगळा **administrator** तयार करा आणि access URL वर त्या account ने या.

---

## 6. Expert साठी छोटे नियम

<div style="background:#fff7ed;padding:10px 14px;border-left:4px solid #ea580c;border-radius:6px">

1. Mic आणि notifications <b>Allow</b>. नाहीतर CCP वर call ऐकू किंवा दिसेना.  
2. Number चा country = instance चा country, तर Free tier DID लागू शकतो. वेगळा देश = telephony charge.  
3. Toll-free आणि DID वेगळे bill आहेत. US शिवाय toll-free free tier मध्ये मोफत नसतो.  
4. Default IVR नमुना आहे. Queue, priority, callback लक्षात ठेवा — पुढे हेच concepts येतील.  
5. ACW संपवायचा एक बटण: <b>Close contact</b>.  
6. Outbound देश कमी दिसले तर region quota पाहा. Quota request शिवाय देश वाढत नाहीत.  
7. Emergency access फक्त lab. खरा admin वेगळा ठेवा. Root रोज वापरू नका.  
8. Agents चा पत्ता access URL आहे. AWS console चा पत्ता नव्हे.

</div>

---

## 7. 30 सेकंदात आठवण

```text
Claim number
  ├── Country + DID किंवा Toll-free
  ├── Mic + notifications Allow
  └── स्वतः call करा → default IVR

CCP = agent चा screen-फोन
  ├── Accept, Hold, Mute, End
  ├── Quick Connects = जतन केलेले transfer
  └── End → After Call Work → Close contact शिवाय नवीन call नाही

Outbound देश = instance region चे service quota
Login
  ├── Access URL        → सामान्य user
  └── Emergency access  → root, फक्त अडचण
```

**पुढचा lecture:** Amazon Connect च्या user interface मधले पर्याय.
