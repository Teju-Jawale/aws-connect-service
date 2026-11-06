# Amazon Connect Instance तयार करणे

> **Course:** Amazon Connect Complete Training Course  
> **या lecture चा विषय:** contact center ची पहिली गोष्ट — **instance**  
> **पुढचा lecture:** phone number claim करणे

<span style="color:#0b6e4f"><b>एक वाक्य:</b></span> **Instance** म्हणजे तुमचा contact center. Region निवडा, नाव द्या, users कुठे राहतील ते सांगा, calls आणि storage ठरवा, मग **Create instance**.

---

## 1. कुठे तयार करायचे

Console → वर **Search** → `Amazon Connect` → **Create instance**.

<div style="background:#fef2f2;padding:10px 14px;border-left:4px solid #dc2626;border-radius:6px">

पान <b>Region unsupported</b> म्हणाले तर account बिघडलेला नाही. तो region मध्ये Connect उपलब्ध नाही.

Video मध्ये <b>Asia Pacific (Mumbai)</b> unsupported दिसले. Instructor ने <b>Asia Pacific (Sydney)</b> निवडले — त्याच्या जवळचा supported region.

</div>

प्रत्येक service प्रत्येक region मध्ये नसते. Supported regions ची यादी त्याच पानावर असते. ती यादी बदलते, त्यामुळे create करण्यापूर्वी यादी पाहा.

तुमचा देश यादीत नसला तरी supported region निवडून Connect वापरता येतो. फोनवरून **international call** लागू शकते. स्वस्त पर्याय: **VoIP** (instructor ला **Skype** आवडते).

<span style="color:#b45309"><b>VoIP अट:</b></span> **outbound caller ID** हवा. म्हणजे call केल्यावर तुमचा **स्थिर phone number** दिसावा. Random number किंवा **Private** चालत नाही — पुढे येणारा number वापरून features बांधणार आहोत.

---

## 2. Identity — users कुठे राहतील

Connect ला सांगा: login खाती कोण सांभाळेल.

| पर्याय | काय होते | कधी वापरावे |
|---|---|---|
| <span style="color:#0b6e4f"><b>1. Connect च्या आत</b></span> | प्रत्येक agent साठी वेगळे account + वेगळा password | हा course. सोपे |
| **2. Active Directory** | Windows चा तोच username / password | कंपनीकडे AD असेल तर |
| <span style="color:#1d4ed8"><b>3. SAML 2.0</b></span> | **Single Sign-On.** Link वर क्लिक, password नाही | सर्वोत्तम daily experience |

**पर्याय 1 ची किंमत:** admin ला प्रत्येक user स्वतः तयार करावा लागतो. User ला Windows शिवाय आणखी एक password लक्षात ठेवावा लागतो.

**SAML चा फायदा:** बरेच identity providers AD शी जोडलेले असतात. माणूस Windows मध्ये login केल्यावर Connect आपोआप उघडतो.

या course मध्ये **पर्याय 1**.

---

## 3. Instance चे नाव = URL

नाव unique हवे. ते URL चा भाग होते.

```text
https://<नाव>.my.connect.aws
```

Video चे उदाहरण: नाव `bpbroadband` → `https://bpbroadband.my.connect.aws`

जागा, चुकीचे spelling नंतर सहज बदलत नाही. छोटे, स्पष्ट नाव द्या. हे agents चे login पान असेल. AWS console चे पान नव्हे.

---

## 4. बाकी सेटिंग्ज (course प्रमाणे)

| पान | काय निवडायचे | का |
|---|---|---|
| Administrator | **No administrator** | पुढच्या video मध्ये console मधून admin म्हणून login |
| Calls | **Incoming + outgoing** दोन्ही | फक्त एक ठेवला तर दुसरी दिशा मिळणार नाही |
| Data storage | **Default** | Connect स्वतः S3 bucket आणि permissions तयार करतो |
| Customer Profiles | **चालू** ठेवा | Free tier मध्ये **1000 profiles** मोफत (video प्रमाणे) |
| शेवट | Review → **Create instance** | काही मिनिटे. यशस्वी पानावर agent login link |

### Storage कुठे जाते

| डेटा | service |
|---|---|
| Call recordings | **S3** |
| Scheduled reports | **S3** |
| Chat transcripts | **S3** |
| Logs | **CloudWatch** |

**Customize Data Storage** केले तर प्रत्येक प्रकार वेगळा सेट करता येतो, आणि **encryption** चालू करता येते. शिकताना default पुरे आहे.

**Customer Profiles** = एका customer ची माहिती आणि contact history एकाच दृश्यात. Agent call दरम्यान हे पाहतो. **Salesforce** सारख्या बाहेरच्या app ची माहितीही जोडता येते.

S3 आणि CloudWatch चे पैसे Connect च्या bill बाहेर असू शकतात (`cost.md`).

---

## 5. Expert साठी छोटे नियम

<div style="background:#fff7ed;padding:10px 14px;border-left:4px solid #ea580c;border-radius:6px">

1. आधी <b>region</b>, मग Connect. Unsupported पान = चुकीचा region.  
2. जवळचा supported region निवडा. Free <b>DID</b> त्या country चा असतो (`cost.md`).  
3. Lab साठी users Connect मध्ये. खरी कंपनी असेल तर <b>SAML 2.0</b> किंवा <b>Active Directory</b>.  
4. Alias अद्वितीय आणि URL चा कायम भाग. तयार करण्यापूर्वी नाव वाचा.  
5. Course अनुसरण करत असाल तर इथे admin तयार करू नका.  
6. Incoming आणि outgoing दोन्ही चालू ठेवा.  
7. Create दाबून पुन्हा दाबू नका. काही मिनिटे लागतात.  
8. चाचणी calls साठी VoIP वापराल तर <b>स्थिर caller ID</b> असलेली service घ्या. Private number पुढचे labs तोडते.  
9. यशस्वी पानावरील link जतन करा. Agents त्या link वर login करतात.

</div>

---

## 6. 30 सेकंदात आठवण

```text
Create instance
  ├── Supported region निवडा
  ├── Identity
  │     ├── Connect users     ← हा course
  │     ├── Active Directory  ← Windows password
  │     └── SAML 2.0          ← SSO, password नाही
  ├── Unique alias → https://alias.my.connect.aws
  ├── Admin skip
  ├── Inbound + outbound
  ├── S3 + CloudWatch (default)
  └── Customer Profiles चालू (1000 free)
```

**पुढचा lecture:** या instance साठी नवीन phone number claim करणे, म्हणजे contact center ला call करता येईल.
