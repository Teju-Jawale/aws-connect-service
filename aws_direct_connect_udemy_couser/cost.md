# Amazon Connect — Pricing (Cost)

> **Course:** Amazon Connect Complete Training Course  
> **या lecture चा विषय:** पैसे कसे लागतात  
> **अधिकृत पान:** [aws.amazon.com/connect/pricing](https://aws.amazon.com/connect/pricing)

<span style="color:#0b6e4f"><b>एक वाक्य:</b></span> Amazon Connect मध्ये <b>setup fee, commitment, minimum monthly fee, agent license, server fee</b> नाही. <b>जे वापराल तेवढेच bill</b>.

---

## 1. पैसे कशासाठी लागतात

| काय | bill? |
|---|---|
| Agent account तयार करणे | <span style="color:#0b6e4f"><b>नाही</b></span> — कितीही agents |
| एकाच वेळी कितीही agents login | <span style="color:#0b6e4f"><b>नाही</b></span> |
| Server / license | <span style="color:#0b6e4f"><b>नाही</b></span> |
| <b>Inbound</b> आणि <b>outbound calls</b> | <span style="color:#c2410c"><b>होय</b></span> |
| S3 (call recording), Lambda, इतर AWS services | <span style="color:#c2410c"><b>वेगळे bill</b></span> |

Agent जास्त असले तरी पैसे <b>usage</b> वर लागतात, headcount वर नाहीत.

---

## 2. तीन प्रकारचे charges

<div style="background:#eff6ff;padding:10px 14px;border-left:4px solid #2563eb;border-radius:6px">

1. <b>Telephony charges</b> — phone number <b>reserve</b> करण्याचे पैसे  
2. <b>Call charges</b> — येणारे (<b>inbound</b>) आणि जाणारे (<b>outbound</b>) calls  
3. <b>Service usage charges</b> — Connect ची सेवा वापरल्याबद्दल

</div>

### Telephony — number चे दोन प्रकार

| | <span style="color:#1d4ed8">DID</span> | <span style="color:#7c3aed">Toll-free</span> |
|---|---|---|
| पूर्ण नाव | Direct Inward Dial | Toll-free number |
| अर्थ | सामान्य phone number | caller ला call चे पैसे लागत नाहीत |
| caller | <b>caller</b> पैसे देतो | <b>company</b> (तुम्ही) पैसे देता |
| Free tier | बहुतेक वेळा होय (खाली पहा) | साधारणपणे <b>नाही</b> |

### Service usage — जगभर <span style="color:#0b6e4f">समान दर</span>

| सेवा | bill कसे |
|---|---|
| Voice | प्रति <b>minute</b> |
| Chat | प्रति <b>message</b> |
| Tasks | प्रति <b>task</b> |

सुरुवातीला फक्त <b>voice</b> होते. नंतर <b>chat</b>, मग आणखी capabilities.

### Telephony + Call charges — <span style="color:#c2410c">country / region नुसार बदलतात</span>

Pricing page वर:

- DID आणि toll-free number ची किंमत  
- देशांचे groups → <b>inbound</b> call rates  
- खाली scroll → <b>outbound</b> call rates (देशानुसार)

---

## 3. Free tier — पहिले 12 महिने

<div style="background:#ecfdf5;padding:10px 14px;border-left:4px solid #059669;border-radius:6px">

Instance <b>ज्या country मध्ये</b> चालतो, त्या country चा <b>1 DID मोफत</b>.

उदाहरण: instance <b>Sydney</b> → Australia चा number मोफत. दुसऱ्या country चा number = full charge.

</div>

| मोफत (दर महिना) | मर्यादा |
|---|---|
| DID | instance च्या country चा <b>1 number</b> |
| Service usage | <b>90 minutes</b> |
| Inbound calls | <b>30 minutes</b> |
| Outbound calls | <b>30 minutes</b> — फक्त <b>त्याच country</b> कडे जिथे instance आहे |

<span style="color:#b45309"><b>लक्षात ठेवा</b></span>

- हे <b>DID</b> आहे, toll-free नाही.  
- <b>अपवाद — US regions:</b> DID <b>किंवा</b> toll-free निवडता येतो.  
- दुसऱ्या country चा number किंवा दुसऱ्या country कडे outbound = charge.  
- अटी आणि दर <b>कधीही बदलू शकतात</b>. कामाआधी pricing page तपासा.

---

## 4. Expert साठी छोटे नियम

<div style="background:#fff7ed;padding:10px 14px;border-left:4px solid #ea580c;border-radius:6px">

1. <b>Region आधी ठरवा.</b> Free DID आणि free outbound त्या country शी बांधलेले आहेत.  
2. नको असलेला number घेऊ नका — telephony हा <b>number धरल्याबद्दल</b> charge आहे.  
3. Toll-free “मोफत call” वाटतो, पण <b>तुमचे bill</b> वाढते. US free-tier शिवाय toll-free free नाही.  
4. Recording = <b>S3</b>. Custom logic = <b>Lambda</b>. हे Connect च्या bill बाहेर आहेत.  
5. Agents मर्यादेत नाहीत. पैसे <b>minutes / messages / tasks</b> मध्ये आहेत.  
6. Quote देताना तीन ओळी सांगा: number + call minutes + service usage. नंतर add-on services.  
7. दर जुने असू शकतात. स्रोत एकच: [pricing page](https://aws.amazon.com/connect/pricing).

</div>

---

## 5. 30 सेकंदात आठवण

```text
Pay as you go
  ├── Telephony   → number reserve (DID / Toll-free) → country नुसार
  ├── Call        → inbound + outbound minutes       → country नुसार
  └── Service     → voice / chat / tasks             → जगभर समान

Free 12 months
  ├── 1 DID (instance country; US = DID किंवा toll-free)
  ├── 90 min service
  ├── 30 min inbound
  └── 30 min outbound (same country only)
```

**पुढचा lecture:** AWS account तयार करणे आणि management console ची सफर. Account आणि console basics आधीच माहित असतील तर तो lecture skip करता येतो.
