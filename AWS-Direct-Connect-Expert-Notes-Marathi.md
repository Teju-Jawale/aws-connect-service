# AWS Direct Connect — तज्ज्ञ अभ्यास नोट्स (एक तासात Expert)

ही नोट्स VPC, subnet आणि basic BGP माहीत असलेल्या network architect साठी आहेत. एक तासात Direct Connect वर expert चर्चा करता यावी, हा उद्देश आहे. विपणन नाही. आकडे AWS User Guide, quotas page, SLA आणि pricing pages वरून घेतले आहेत. डॉलर दर बदलतात; port-hour चे दर इथे मुद्दाम दिलेले नाहीत.

ठळक केलेले शब्द पाठ करा. बाकी वाक्य समजून घ्या.

---

## 60 मिनिटांची योजना

| वेळ | काय वाचा | तास संपल्यावर काय बोलता आले पाहिजे |
| --- | --- | --- |
| **0–10** | विभाग 1–3 | Direct Connect काय आहे, काय नाही; fiber path; dedicated vs hosted speeds |
| **10–25** | विभाग 4–5 | Private, public, transit VIF; DX Gateway काय जोडतो आणि काय transit करत नाही |
| **25–40** | विभाग 6 | BGP quotas, communities `7224:9100/9200/9300` आणि `8100/8200`, failover |
| **40–50** | विभाग 7–11 | 99.99 / 99.9 / 95 SLA अटी, LAG, MACsec, SiteLink, quota table |
| **50–60** | विभाग 16–17, शेवटचा memory sheet, नंतर Q&A | Build order, decision table, trap answers. नवे मुद्दे या दहा मिनिटांत उघडू नका |

50 व्या मिनिटानंतर फक्त memory sheet आणि Q&A. Deep notes परत उघडू नका.

---

## 1. Direct Connect काय आहे

**Direct Connect** म्हणजे ग्राहकाच्या network पासून AWS पर्यंतचा **private dedicated network path**. Traffic तुमच्या ISP च्या public internet मधून जात नाही. एक टोक customer router किंवा firewall वर असतो. दुसरे टोक Direct Connect location मधील AWS router वर असते. तिथून traffic AWS backbone वरून Region मध्ये जातो.

Physical link आला म्हणजे VPC access आला, असे नाही. VPC, public AWS services, किंवा Transit Gateway पर्यंत पोहोचण्यासाठी **VIF** आणि योग्य gateway लागते. VIF शिवाय port फक्त fiber आहे.

डीफॉल्टने हा path **encrypt होत नाही**. Customer link वर encryption हवी असेल तर **MACsec**, Layer 3 ला **IPsec**, आणि application ला **TLS**. तीन वेगळे स्तर आहेत. एक दुसऱ्याला बदलत नाही.

MACsec पानावर AWS असे लिहिते: PoP आणि Region मधील AWS network वर physical layer encryption आहे. ही AWS ची अंतर्गत संरक्षण आहे. ती customer-managed encryption नाही. Compliance ला customer key हवी असेल तर MACsec, VPN, किंवा TLS लागते.

### काय निवडावे

| गरज | योग्य पर्याय | लक्षात ठेवा |
| --- | --- | --- |
| स्थिर latency, उच्च steady throughput, on-prem ते AWS | **Direct Connect** dedicated किंवा hosted | Port-hour idle असतानाही लागतो. Encryption डीफॉल्ट नाही |
| लवकर, कमी bandwidth, encrypted path | **Site-to-Site VPN** | Internet path. DX चा backup म्हणून सामान्य |
| फक्त internet वर public API | Public internet | Latency बदलते. Dedicated port नाही |
| VPC च्या आत AWS किंवा partner service, private IP | **PrivateLink** (interface endpoint) | हा circuit नाही. On-prem ला तो service हवा असेल तर आधी DX किंवा VPN ने VPC मध्ये या |
| अनेक VPC, खाती, VPN एका Regional router वर | **Transit Gateway** | TGW fiber देत नाही. On-prem साठी transit VIF + DX Gateway |
| हाताने VIF आणि BGP नको; last-mile partner ने managed जोडणी | **AWS Interconnect – last mile** | Attach point DX Gateway. चार links, दोन facilities, ECMP. Port पर्यंत 99.99% SLA. Bandwidth आणि Region current docs वर पहा |
| AWS VPC ते दुसऱ्या cloud चा VPC, private | **AWS Interconnect – multicloud** | AWS बाजूचा attach point DX Gateway. Provider आणि Region pairs बदलतात |

Interconnect हा Direct Connect चा बदली शब्द नाही. Last mile आणि multicloud दोन्ही DX Gateway ला चिकटतात. Classic Direct Connect मध्ये तुम्ही connection, VIF आणि BGP स्वतः डिझाइन करता.

PrivateLink आणि Direct Connect एकत्र राहू शकतात. Direct Connect traffic VPC पर्यंत आणतो. PrivateLink त्या VPC मधून service देतो.

---

## 2. Physical architecture

मार्ग हा आहे:

`customer router/firewall → cross-connect (DX location / colocation) → AWS Direct Connect endpoint → AWS backbone → Region`

**LOA-CFA** म्हणजे Letter of Authorization आणि Connecting Facility Assignment. Dedicated connection मंजूर झाल्यावर हे पत्र मिळते. Colocation operator या पत्रावरून तुमच्या cage पासून AWS panel पर्यंत **cross-connect** बसवतो. Cross-connect कोण मागतो: ग्राहक, किंवा ग्राहकाचा network provider, facility कडे. AWS तुमच्या cage मध्ये fiber ओढत नाही.

Dedicated connection साठी provider हा **DX Partner असणे आवश्यक नाही**. कोणताही network provider fiber देऊ शकतो. **Hosted** connection मात्र फक्त **AWS Direct Connect Partner** कडून येते. Console मधून hosted connection मागता येत नाही.

Fiber **single-mode** आहे. Optics, quotas पानाप्रमाणे:

| Port | Optic |
| --- | --- |
| Dedicated 1 Gbps | **1000BASE-LX (1310 nm)** |
| Dedicated 10 Gbps | **10GBASE-LR (1310 nm)** |
| Dedicated 100 Gbps | **100GBASE-LR4** |
| Dedicated 400 Gbps | **400GBASE-LR4** |

100G आणि 400G साठी docs एकच wavelength लिहीत नाहीत. LR4 हे नाव पाठ करा. 1310 nm फक्त 1G आणि 10G च्या LX/LR साठी docs मध्ये आहे.

1 Gbps किंवा त्याहून जलद connection साठी आणखी अटी:

- संपूर्ण मार्गावर **802.1Q** VLAN pass झाला पाहिजे. मधला switch VLAN काढून टाकत असेल तर VIF खाली राहतो.
- Router ला **BGP** आणि **BGP MD5** हवा.
- Dedicated port वर auto-negotiation कधी चालू आणि कधी बंद, हे त्या DX endpoint वर अवलंबून असते. Connection `available` आणि VIF `down` असेल तर Layer 2 तपासा.
- **BFD** ऐच्छिक आहे. AWS बाजूला asynchronous BFD प्रत्येक VIF वर आपोआप enabled असतो. तो तेव्हाच काम करतो, जेव्हा तुम्ही router वर BFD configure करता.

Direct Connect location एका associated Region शी जोडलेले असते. Public VIF वरून एका public Region मधील connection इतर public Regions मधील public AWS services पर्यंत जाऊ शकतो. Private access साठी VIF प्रकार आणि gateway ठरवतात, फक्त location नाही.

---

## 3. Connection प्रकार आणि speeds

### Dedicated

**Dedicated** connection म्हणजे एक Ethernet port **एका ग्राहकासाठी**. Speeds: **1, 10, 100, 400 Gbps**.

तुम्ही AWS console मधून मागता. Port-hour त्या connection च्या owner account ला लागतो. याच port वर अनेक VIF असू शकतात, quota विभाग पाहा.

### Hosted

**Hosted** connection मध्ये **Partner** physical port चा मालक असतो. तुम्हाला त्या port चा एक capacity तुकडा मिळतो.

नेमक्या speeds:

**50, 100, 200, 300, 400, 500 Mbps, 1, 2, 5, 10, 25 Gbps.**

- 1, 2, 5, 10, 25 Gbps फक्त त्या Partners कडून, ज्यांनी AWS च्या specific requirements पूर्ण केल्या आहेत.
- **25 Gbps** फक्त तिथे, जिथे location ला 100 Gbps ports आहेत.
- Speed बदल फक्त Partner करू शकतो. Support असेल तर connection delete करून नव्याने बनवावी लागत नाही.
- AWS traffic **police** करतो. ठरवलेल्या दरानंतरचे packets **drop** होतात. Burst traffic ची throughput गुळगुळीत traffic पेक्षा कमी दिसू शकते. Partner लाही तेच policing करणे अपेक्षित आहे.
- Accept केल्याशिवाय VIF बनवता येत नाही.

### Hosted connection आणि hosted virtual interface

हे दोन शब्द गोंधळवतात. चर्चा अशी उघडा:

| मॉडेल | कोण मालक | तुम्ही काय करता | VIF |
| --- | --- | --- | --- |
| **Hosted connection** (सध्याचा Partner capacity मॉडेल) | Partner physical port; accept नंतर connection तुमच्या account मध्ये | Connection accept करा, मग VIF तयार करा | **फक्त एक** VIF: private, public, **किंवा** transit |
| **Hosted virtual interface** (docs चा सध्याचा cross-account अर्थ) | Connection तुमच्या account मध्ये राहतो | दुसऱ्या account ID वर VIF तयार करता; तो account accept करतो | Private किंवा public; transit ही dedicated/hosted **connection** सुविधा आहे |
| Legacy Partner VIF | जुना Partner-allocated VIF. SLA अजून “hosted virtual interfaces” ला dedicated SLA मधून वगळतो | जुना VIF अजून असेल तर console मध्ये प्रकार तपासा | Transit साठी समर्थित वस्तु dedicated connection किंवा hosted connection आहे |

एक वाक्य: **hosted connection वर VIF एक, आणि तो private, public किंवा transit असू शकतो. Dedicated port तुमचा. Hosted port Partner चा.**

---

## 4. Virtual interfaces

VIF शिवाय Direct Connect व्यवसाय traffic नेत नाही. तीन प्रकार पाठ करा.

### Private VIF

**Private VIF** Amazon VPC ला **private IP** ने जोडतो.

दोन जोडण्या:

- त्याच Region मधील **virtual private gateway (VGW)** ला थेट. एक private VIF, एक VPC.
- **Direct Connect gateway** ला. मग त्या DX Gateway शी associated VGW इतर Regions मध्ये असू शकतात. **AWS China Regions यात येत नाहीत.**

Private VIF थेट Transit Gateway ला लागत नाही. अनेक VPC आणि TGW हवे असतील तर **transit VIF**.

AWS बाजूला दिसणारे routes म्हणजे associated VPC चे CIDR. ग्राहक on-prem prefixes advertise करतो. हे prefixes public किंवा private असू शकतात.

### Public VIF

**Public VIF** AWS चे **public** services घेतो: S3 public endpoint, DynamoDB public endpoint, public API, EC2 चे public IP, Amazon.com. Non-Amazon prefixes मिळत नाहीत.

BGP peering साठी **तुमच्या मालकीचे public IP** लागतात. **EIP** किंवा Amazon pool मधील **BYOIP** peering साठी वापरू नका. पर्याय:

- तुमचा public IPv4 CIDR. दोन्ही peers वर **तोच subnet mask**. `/31` चालतो.
- Partner किंवा ISP चा range, **LOA-CFA** सह.
- AWS कडून `/31` मागावा लागेल. Support केस आणि use case. AWS हे पूर्ण करेलच असे नाही.

IPv6 peering address तुम्ही स्वतः देत नाही. AWS **`/125`** देते.

किमान **एक** prefix advertise करावा लागतो. कमाल **1,000**. ही मर्यादा वाढत नाही. IPv4 prefix length `/1` ते `/32`. IPv6 `/1` ते `/64`.

AWS Amazon public prefixes advertise करतो. `ip-ranges.json` पेक्षा aggregation वेगळी असू शकते. तुमचे BYOIP त्या JSON मध्ये नसले तरी public VIF वर AWS ती addresses advertise करतो. China prefixes फक्त China Regions मध्ये. Commercial prefixes फक्त commercial Regions मध्ये.

ग्राहकाने दिलेले prefixes AWS बाहेर पुन्हा advertise होत नाहीत: इतर Direct Connect customers, AWS Global Network चे peers, Amazon चे transit providers. AWS network च्या आत मात्र public VIF वरील prefixes इतर AWS customers ला दिसू शकतात. त्यामुळे firewall filter ठेवा.

**Jumbo frames public VIF वर नाहीत. MTU 1500.** Jumbo चा दस्तऐवज private आणि transit पुरता मर्यादित आहे.

Public VIF तयार होताना state अनेकदा **`verifying`** असतो. AWS ला prefix ची मालकी तपासायची असते. Approve शिवाय traffic चालत नाही.

### Transit VIF

**Transit VIF** एका किंवा अधिक **Transit Gateway**, किंवा **Cloud WAN** core network पर्यंत जातो, जे त्या **DX Gateway** शी associated आहेत.

- Dedicated किंवा hosted connection.
- कोणतीही speed.
- VIF **DX Gateway** ला लागतो. **TGW ला थेट नाही.**

TGW association वर **allowed prefixes** द्यावे लागतात. On-prem कडे AWS फक्त ते prefixes advertise करतो. VPC CIDR आपोआप जात नाही. हे private VIF + VGW पेक्षा वेगळे आहे. तपशील विभाग 5 मध्ये.

### VLAN

- **802.1Q**, tag **1–4094**.
- त्या connection वर unique.
- तयार केल्यानंतर **बदलता येत नाही**.
- Hosted connection वर VLAN **Partner** देतो.

### Address family

एका VIF वर **IPv4 BGP**, **IPv6 BGP**, किंवा **दोन्ही (dual-stack)**. एकाच address family चे दोन BGP sessions एका VIF वर चालत नाहीत.

### दुसऱ्या account ची VIF

Connection owner दुसऱ्या AWS account साठी VIF तयार करू शकतो. तो account accept करेपर्यंत state **`confirming`**. नाकारल्यास **`rejected`**.

**Port-hour connection owner देतो.** Data transfer out चा हिशोब traffic च्या resource owner कडे जातो. नेमका नियम विभाग 12 मध्ये आहे. VIF तयार केल्याबद्दल वेगळे port charge नाही.

### BGP MD5

AWS **MD5 डीफॉल्टने चालू** ठेवतो. हा पर्याय बंद करता येत नाही. Key तुम्ही द्या, किंवा AWS तयार करेल. API नुसार key ची लांबी 6 ते 80 characters.

### ASN नियम

- Customer gateway ASN आणि VGW किंवा DX Gateway ASN **समान असू शकत नाहीत**. समान ASN असेल तर BGP येत नाही.
- **तोच customer ASN** अनेक VIF वर वापरता येतो.
- **तोच AWS ASN** वेगवेगळ्या connections वरील VIF वर वापरता येतो.
- उदाहरण: VGW ASN 64512, customer ASN 64511, दोन वेगळ्या connections वर. हे वैध आहे.

Public VIF वर Amazon बाजूचा peering ASN **7224** असतो. Private VIF थेट VGW ला असेल तर Amazon ASN तो VGW चा ASN असतो. Private VIF DX Gateway ला असेल तर Amazon ASN तो **DX Gateway चा ASN** असतो.

### MTU आणि jumbo

| VIF | MTU |
| --- | --- |
| Public | **1500**. Jumbo नाही |
| Private | **1500** किंवा **9001** |
| Transit | **1500** किंवा **8500** |

Link layer frame **1522** किंवा **9023** bytes: 14 Ethernet + 4 VLAN + IP datagram + 4 FCS.

Jumbo पहिल्यांदा चालू केल्यास खालील physical connection update होऊ शकतो. त्या connection वरील **सर्व VIF** सुमारे **30 seconds** साठी खाली जाऊ शकतात.

Hosted connection वर jumbo तेव्हाच, जेव्हा parent connection वर आधीच jumbo enabled असेल.

दोन private VIF एकच route वेगवेगळ्या MTU ने देतात, किंवा तोच route VPN ही देतो, तर **1500** वापरले जाते. Transit Gateway वर jumbo **8500** पर्यंत. Jumbo propagated Direct Connect routes आणि Transit Gateway static routes ला लागू होतो. EC2 प्रकार C1, CC1, T1, M1 jumbo support करत नाहीत; ते मोठे frames टाकतात.

`/31` सर्व VIF प्रकारांवर चालते. Private आणि transit peering IP कोणत्याही वैध range मधील असू शकतो, अगदी customer-owned public IP सुद्धा, पण तो फक्त peering साठी. तो advertise करू नका आणि NAT साठी वापरू नका.

---

## 5. Direct Connect Gateway

**DX Gateway** हा **global** resource आहे. तो एखाद्या Region मध्ये “बसत” नाही. Commercial partition आणि **AWS GovCloud (US)** याच्याशी तो काम करतो. **AWS China Regions यात येत नाहीत.** China वेगळा partition आहे. एक commercial DX Gateway China VPC ला जोडत नाही.

AWS याला data path मधील एक regional box म्हणत नाही. Docs चे वर्णन: distributed BGP route reflectors चा संच, data traffic path च्या बाहेर, त्यामुळे एका Region वर अवलंबून राहावे लागू नये. तरीही तुमचा traffic on-prem पासून DX location आणि backbone मार्गे लक्ष्य VPC किंवा TGW कडे जातो. DX Gateway म्हणजे दुसरा Transit Gateway नाही.

ASN तयार करताना द्यावा लागतो. Private range: **64512–65534** किंवा **4200000000–4294967294**. Default **64512**. VIF जेव्हा DX Gateway ला लागतो, तेव्हा Amazon बाजूचा BGP ASN हाच असतो. Cloud WAN सोबत वापराल तर हा ASN core network च्या ASN range मध्ये असू नये.

### कोटा

| वस्तू | कोटा | वाढ |
| --- | --- | --- |
| DX Gateway per account | **200** | Soft. SA किंवा TAM मार्गे विनंती |
| VGW per DX Gateway | **20** | वाढत नाही |
| Transit Gateway per DX Gateway | **6** | वाढत नाही |
| Private किंवा transit VIF per DX Gateway | **30** | वाढत नाही |
| DX Gateway per Transit Gateway | **20** | वाढत नाही |
| VIF per VGW | मर्यादा नाही | — |

### VGW association

Private VIF → DX Gateway → अनेक VGW, अनेक Regions (China वगळता), अनेक accounts.

Allowed prefix येथे **filter** आहे. यादी VPC CIDR एवढी किंवा त्याहून **रुंद** हवी. AWS ग्राहकाला VPC CIDR advertise करतो, तुम्ही लिहिलेला वेगळा prefix नाही.

उदाहरण: VPC `10.0.0.0/16`.

- Allowed `10.0.0.0/15` असेल तर तुम्हाला `10.0.0.0/16` मिळतो.
- Allowed `10.0.0.0/24` किंवा असंबंधित `22.0.0.0/24` असेल तर route मिळत नाही.

### TGW association

Transit VIF → DX Gateway → TGW association.

येथे allowed prefix filter नाही. **यादीत जे लिहाल तेच** on-prem कडे advertise होते, DX Gateway च्या ASN ने.

उदाहरण: VPC `10.0.0.0/16` TGW ला असेल आणि allowed `10.0.0.0/8` असेल, तर on-prem ला **`10.0.0.0/8`** दिसतो, `/16` आपोआप नाही.

एका DX Gateway वर अनेक TGW असतील तर allowed prefixes **overlap** करू शकत नाहीत. एका TGW कडे `0.0.0.0/0` आणि दुसऱ्याकडे आणखी prefix ठेवता येत नाही, कारण `0.0.0.0/0` सर्वांना व्यापते.

### DX Gateway काय transit करत नाही

साधा नियम, आधी हा बोला: **DX Gateway दोन VPC एकमेकांशी जोडत नाही. On-prem ते VPC. VPC ते on-prem.**

दोन VGW एकाच DX Gateway वर असले तरी त्यांच्यामध्ये DX Gateway router बनत नाही.

**अपवाद, November 2021 पासून docs मध्ये:** एकाच VIF वरून on-prem असा supernet advertise करतो जो त्या VPC CIDR ला overlap करतो, उदाहरणार्थ on-prem `10.0.0.0/8` आणि VPC `10.0.0.0/24` तसेच `10.0.1.0/24`. तेव्हा VPC एकमेकांशी Direct Connect endpoint मार्गे बोलू शकतात. हे डिझाइन नाही. हे अपघात आहे.

थांबवण्याचे मार्ग:

- Overlap होणारा supernet advertise करू नका. अधिक specific on-prem routes द्या.
- Security groups ने VPC मधील परस्पर traffic थांबवा.
- ज्या VPC मध्ये परस्पर traffic नको, त्यांच्यासाठी वेगळे DX Gateway.

TGW वापरल्यास VPC मधील transit **TGW** करतो, DX Gateway नाही. त्याच DX Gateway वर अनेक TGW असतील आणि on-prem supernet association prefixes ला cover करत असेल, तर traffic मागे येऊ शकतो. नको असेल तर TGW route table मध्ये **blackhole**.

एक वाक्य नक्की बोला: **VGW association ने VPC-to-VPC transit मिळत नाही. Transit VIF + DX Gateway + TGW ने on-prem अनेक VPC पर्यंत जातो, आणि VPC मधील जटिल routing TGW करतो.**

Cross-account: VGW किंवा TGW चा owner association proposal पाठवतो. DX Gateway चा owner accept करतो आणि allowed prefixes ठेवू शकतो. Multi-account DX Gateway वापरल्याबद्दल वेगळे Direct Connect charge docs मध्ये नाही. TGW चे data processing आणि attachment charges मात्र लागू राहतात.

---

## 6. BGP

Direct Connect वर ग्राहक router आणि AWS यांच्यात **eBGP** असतो. iBGP नाही.

### कोणता ASN दिसतो

| जोडणी | Amazon बाजू |
| --- | --- |
| Public VIF | **7224** |
| Private VIF थेट VGW | त्या VGW चा ASN. न दिल्यास default **64512**. VGW तयार केल्यानंतर ASN बदलत नाही |
| Private किंवा transit VIF DX Gateway वर | DX Gateway चा ASN. न दिल्यास default **64512** |

Customer ASN range: 1 ते 4294967294, reserved वगळून. Private 16-bit **64512–65534**. Private 32-bit **4200000000–4294967294**.

Public VIF वर public ASN असेल तर तो तुमच्या मालकीचा हवा; AWS तपासतो. Private ASN वापरल्यास AWS तुमचे prefixes पुढे advertise करताना तो ASN काढून **7224** घालतो. त्यामुळे public VIF वर **private ASN prepending बाहेरच्या जगतात काम करत नाही**. Public ASN prepending दिसते.

### Timer आणि TTL

| Setting | मूल्य |
| --- | --- |
| Default hold | 90 seconds |
| Minimum hold | 3 seconds. 0 चालत नाही |
| Default keepalive | 30 seconds |
| Minimum keepalive | 1 second |
| Graceful restart | 120 seconds. **BFD सोबत वापरू नका** |
| BFD minimum interval | **300 ms** |
| BFD minimum multiplier | **3** |
| BGP TTL | **1**. Multihop नाही. दुसरा TTL नाही |

Timer दोन बाजूंच्या नीच मूल्यापर्यंत खाली येतात. BFD मात्र हळू दर निवडतो: प्रत्यक्ष interval हा local DesiredMinTx आणि peer च्या RequiredMinRx यांच्यातील मोठा आकडा. दिशा स्वतंत्र असू शकते. Detection time = negotiated interval × local multiplier. दोन्ही बाजू minimum वर असतील तर 300 ms × 3 = **900 ms**. हे 90 second hold पेक्षा खूप जलद आहे.

AWS बाजूला asynchronous BFD आधीच enabled असतो. Router वर configure केल्याशिवाय session वर त्याचा परिणाम नाही.

### Route quotas

जास्त prefixes दिल्यास private किंवा transit VIF वर BGP **idle** होतो आणि status **DOWN** दिसतो. Session “स्थापन होऊन शांत” राहत नाही.

| दिशा | कोटा |
| --- | --- |
| On-prem → AWS, private किंवा transit, IPv4 | Default **100**. Prefix controls ने **1,000** पर्यंत |
| On-prem → AWS, private किंवा transit, IPv6 | Default **100**. Prefix controls ने **1,000** पर्यंत |
| Public VIF, एकूण prefixes | **1,000**. वाढत नाही |
| TGW → on-prem, transit VIF | **200**, IPv4 आणि IPv6 **मिळून**. वाढ मागता येते |
| Cloud WAN core network, DX Gateway attachment → on-prem | **5,000**. त्या DX Gateway वरील **सर्व** transit VIF ला ते prefixes मिळतात |
| SiteLink prefix controls | IPv4 **1,000** आणि IPv6 **1,000** पर्यंत |

हवे तेवढेच prefixes द्या. Summarize करा. अधिक specific route तेव्हाच, जेव्हा longest prefix match ने एखादा path मुद्दाम निवडायचा असेल. उगीच `/32` पसरवल्यास quota संपते आणि failover कठीण होते.

### AWS traffic कसे निवडते, private आणि transit

प्राधान्य:

1. Longest prefix match.
2. Local preference.
3. AS path length.
4. MED. AWS MED ला प्राथमिक साधन म्हणत नाही.
5. नंतर **ECMP**, जेव्हा prefix length, local preference, AS path length आणि BGP attributes जुळतात.

Direct Connect user guide स्पष्ट लिहितो: ECMP साठी AS path मधील **ASN समान असणे आवश्यक नाही**. लांबी आणि attributes जुळली की पुरे.

Transit Gateway च्या पानावर वेगळी वाक्ये आहेत: BGP Multipath AS-Path Relax support नाही, आणि DX Gateway attachment वर ECMP तेव्हा जेव्हा prefix, prefix length आणि **AS path** समान असतात. एकाच DX Gateway वर अनेक transit VIF मध्ये ECMP होऊ शकतो. चर्चा करताना थर सांगा. VIF स्तर: length आणि attributes. TGW स्तर: AS path समान ठेवा, वेगळ्या ASN वर ECMP गृहित धरू नका.

समान local preference community नसेल तर AWS associated Region चा DX location पसंत करतो. SiteLink enabled असेल तर हा नियम बदलतो: Region कमी AS path length पसंत करतो, location कोणत्या Region शी associated आहे याकडे दुर्लक्ष करून.

### Local preference communities, private आणि transit

ग्राहकाने on-prem prefixes वर ही communities लावायची. AWS कडून येणारा return traffic यावरून वळतो. Extended communities AWS काढून टाकू शकतो. ही tags एकमेकांशी सोबत वापरू नका.

| Community | अर्थ |
| --- | --- |
| **7224:7100** | Low |
| **7224:7200** | Medium |
| **7224:7300** | High |

High प्राथमिक. Local preference AS path च्या आधी तपासले जाते. Active/active साठी दोन्ही VIF वर **तीच** community, उदाहरणार्थ `7224:7200`. Active/passive साठी primary वर `7224:7300` आणि backup वर `7224:7100`.

### Public VIF communities

येथे गोंधळ जास्त होतो. दिशा पाठ करा.

**तुम्ही AWS कडे** prefixes देताना scope:

| Community | कुठपर्यंत |
| --- | --- |
| **7224:9100** | फक्त local AWS Region |
| **7224:9200** | त्या खंडातील Regions: North America, Asia Pacific, किंवा Europe / Middle East / Africa |
| **7224:9300** | Global, सर्व public Regions |
| काहीच community नाही | **Global**. हे default आहे |

`7224:1` ते `7224:65535` Direct Connect राखून ठेवतो.

**AWS तुमच्याकडे** prefixes देताना:

| Tag | अर्थ |
| --- | --- |
| **7224:8100** | त्या DX PoP च्या associated Region मधून आलेले routes |
| **7224:8200** | त्याच खंडातून आलेले routes |
| Tag नाही | इतर खंड |
| **NO_EXPORT** | AWS जे public prefixes देते, त्या **सर्वांवर** ही well-known community असते |

**NO_EXPORT म्हणजे 7224:9100 नाही.** `7224:9100` local Region scope आहे, customer ते AWS. `NO_EXPORT` वेगळी well-known community आहे. AWS ने दिलेले prefixes तुमच्या network च्या बाहेर internet routing table मध्ये जाऊ नयेत.

इतर public VIF नियम:

- AWS prefixes किमान AS path length **3** ने advertise करतो.
- दोन Regions मधून तेच prefixes, समान attributes आणि समान prefix length असतील तर outbound साठी **home Region** प्राधान्य.
- Inbound load-share साठी समान path attributes ठेवा.
- Transitive routing दोन connections मध्ये support नाही. Traffic Amazon public prefixes कडे गेला पाहिजे.
- AWS source address तपासतो. Source तुमच्या advertised prefix मधला हवा.
- समान prefix internet आणि public VIF दोन्हीकडे असेल तर Direct Connect वर **अधिक specific** द्या, नाहीतर longest match internet कडे जाऊ शकतो.

### AS path loop

Private आणि transit वर AWS मानक eBGP loop prevention वापरतो. AS path मध्ये **7224 दोनदा** आला तर route बसवला जात नाही. हे तेव्हा होते, जेव्हा एका VIF वर AWS कडून शिकलेला route दुसऱ्या VIF वर परत दिला जातो. AWS कडून आलेले routes AWS कडे परत advertise करू नका.

### Active/active, LAG आणि VPN

- **LAG** LACP ने एका endpoint वर bandwidth वाढवतो.
- **BGP ECMP** वेगळ्या VIF किंवा वेगळ्या paths वर traffic वाटतो.
- हे एकच साधन नाही. LAG आला म्हणजे दुसरा location आला, असे नाही.
- Customer बाजूने primary DX ला **local preference** द्या.
- AWS कडून येणारा traffic वळवण्यासाठी private/transit वर `7224:7300` / `7224:7100`, किंवा AS path prepending.
- Public VIF वर prepending फक्त public ASN सोबत बाहेर उपयोगी. Scope साठी `9100/9200/9300`.
- VPN backup असेल तर DX वर **BFD** ठेवा, म्हणजे DX गेल्यावर 90 second hold ची वाट पाहू नये.
- समान prefix VGW वर Direct Connect आणि Site-to-Site VPN दोन्हीकडून आला तर डिझाइन DX ला प्राथमिक ठेवते. सिद्ध करायचे असेल तर failover test करा. अधिक specific route VPN वर चुकून राहिला तर longest match DX ला हरवतो. हा सामान्य दोष आहे.

### दोन्ही दिशा

- Private VIF: AWS VPC CIDR देते. ग्राहक on-prem prefixes देतो.
- Public VIF: AWS Amazon public prefixes देतो. ग्राहक फक्त स्वतःचे public prefixes देतो.
- Transit VIF: AWS allowed prefixes देतो. ग्राहक on-prem prefixes देतो.

---

## 7. Resiliency Toolkit आणि SLA

SLA तीन आहेत. Toolkit चे नाव आणि करारातील Deployment वेगळे शब्द असू शकतात. करार हा अंतिम आहे. SLA शेवटचे अद्यतन या नोट्स लिहिताना **24 July 2026** असे दिसले.

Service credit हा भविष्यातील Direct Connect देयकावर जातो. तो त्या connection च्या **port-hour** charges चा टक्का असतो. Refund नाही. एक डॉलरपेक्षा कमी credit दिला जात नाही. दावा दुसऱ्या billing cycle च्या शेवटापर्यंत Support case ने करावा लागतो. Subject: `Direct Connect SLA Credit Request`.

**Unavailable** म्हणजे त्या Endpoint कडे सर्व संबंधित connections वरून, किंवा single connection च्या बाबतीत त्या connection वरून, डेटा पाठवता किंवा घेता **किमान 120 seconds** येणे.

### तीन SLA

| SLA | मासिक लक्ष्य | किमान रचना |
| --- | --- | --- |
| **Multi-Site Redundant** | **99.99%** | किमान **चार** dedicated connections, किमान **दोन** DX locations, **प्रत्येक** location मध्ये किमान दोन, प्रत्येक connection **वेगळ्या AWS device** वर (console मधील AWS Device ID), सर्व त्याच Endpoint ला, private endpoint **दोन किंवा अधिक AZ** मध्ये, **Enterprise Support**, आणि AWS Solutions Architect सोबत **Well-Architected Review** |
| **Multi-Site Non-Redundant** | **99.9%** | किमान **दोन** connections, किमान **दोन** locations, त्याच Endpoint, private endpoint दोन किंवा अधिक AZ, **Enterprise Support** |
| **Single Connection** | **95%** | एक dedicated connection. Production साठी AWS हे सुचवत नाही |

Credit पायऱ्या, थोडक्यात:

- 99.99 मॉडेल: 99.99 खाली आणि 99.0 पर्यंत 10%; 99.0 खाली आणि 95.0 पर्यंत 25%; 95.0 खाली 100%.
- 99.9 मॉडेल: 99.9 खाली आणि 99.0 पर्यंत 10%; पुढे वरीलप्रमाणे 25% आणि 100%.
- 95 मॉडेल: 95.0 खाली आणि 92.5 पर्यंत 10%; 92.5 खाली आणि 90.0 पर्यंत 25%; 90.0 खाली 100%.

### SLA मिळत नाही असे काय

- Hosted connection.
- Partner कडून आलेला hosted virtual interface.
- Dedicated connection तुमच्या account च्या मालकीचा असेल, आणि Partner फक्त fiber देत असेल, तर SLA लागू होऊ शकतो. मालकी तुमची हवी.
- **LAG मधील ports मिळून एक Connection मोजले जातात.** चार ports चा एक LAG म्हणजे 99.99 साठी चार connections नाही.
- Demarcation पलीकडील last-mile, तुमचे उपकरण, चुकीचे security group, चुकीचे VPC routing, encryption key बंद केल्यास. हे exclusion आहे.
- एक connection अनेक Deployments मध्ये असेल तर त्याचा downtime एका Deployment कडे मोजला जातो. Deployment उपलब्ध राहिला तरी त्या एका connection साठी Single Connection SLA मागता येऊ शकतो.

### Toolkit मधील मॉडेल

| Toolkit मॉडेल | काय बनवतो | SLA शी नाते |
| --- | --- | --- |
| **Maximum resiliency** | एकापेक्षा जास्त locations, वेगळ्या devices वर वेगळ्या connections. आकृतीत प्रत्येक data center चे दोन्ही connections त्याच DX locations कडे जाऊ शकतात. इच्छेनुसार वेगळे locations | 99.99%, वरच्या सर्व अटी पूर्ण झाल्यास |
| **High resiliency** | अनेक locations कडे एकल connections | 99.9%, अटी पूर्ण झाल्यास |
| **Development and test** | **एका** location मध्ये वेगळ्या devices वर वेगळ्या connections. Device failure सहन होते. Location failure सहन होत नाही | 99.9 किंवा 99.99 चा दावा नाही. Non-critical |
| **Classic** | Toolkit शिवाय आणखी connection, ज्यांच्याकडे आधीच connections आहेत | सुमारे **95%**. Redundancy देत नाही |

Toolkit आणखी काय करते:

- Redundant connections ची speed समान ठेवते.
- नावे आपोआप देते.
- ओळखीचा Partner आणि अस्तित्वातले AWS account असेल तर connection approve होते आणि **LOA लगेच** उतरवता येतो.
- नवा ग्राहक किंवा Other partner असेल तर support ticket.
- ऑर्डरचा सारांश: साध्य होणारा SLA आणि port-hour खर्च.
- 1, 10, 100, 400 Gbps शिवाय दुसरी speed निवडल्यास **LAG** तयार करते.
- दोन connections एकाच AWS device वर जाऊ देत नाही.
- **Failover test**: AWS BGP session खाली आणतो. Traffic दुसऱ्या VIF वर गेला का, ते सिद्ध करता. त्या वेळी VIF state **`testing`**.
- CloudWatch metrics.

एक connection वर 95% ऐकू येते. Expert उत्तर मात्र असे असते: **दोन DX locations, वेगळा last-mile, वेगळे customer routers, वेगळे AWS devices, BFD, आणि failover test.** 95% हा production डिझाइन नाही.

### VPN backup

Site-to-Site VPN जलद निघतो आणि IPsec देतो. Bandwidth आणि latency internet वर अवलंबून असतात. DX ची port speed समर्पित असते.

सामान्य डिझाइन: DX प्राथमिक, VPN दुय्यम.

- Customer router वर DX ला जास्त local preference.
- AWS कडे समान prefix दोन्ही मार्गांनी जाऊ द्या, DX वर चांगले attributes.
- DX VIF वर BFD.
- VPN वर चुकून अधिक specific prefix ठेवू नका.
- Public VIF वर VPN, AWS च्या public VPN endpoint पर्यंत, हा एक मार्ग आहे.
- Private IP VPN over Direct Connect हा दुसरा मार्ग आहे, जेव्हा tunnel DX च्या private path वर हवा असेल.

Failover न तपासलेला backup backup नसतो. Toolkit चा BGP failover test वापरा.

---

## 8. LAG

**LAG** म्हणजे Link Aggregation Group. Protocol **LACP**. अनेक dedicated connections एका logical connection सारखे दिसतात. सर्व members **active/active**.

नियम:

- फक्त **dedicated** connections. Speed **1, 10, 100 किंवा 400 Gbps**.
- LAG मधील सर्वांची bandwidth समान.
- **100 Gbps किंवा 400 Gbps: जास्तीत जास्त 2** connections.
- **100 Gbps पेक्षा कमी: जास्तीत जास्त 4**.
- सर्व members **त्याच Direct Connect endpoint** वर. **MLAG support नाही.** AWS अनेक chassis मिळून एक LAG देत नाही.
- त्यामुळे LAG बहुतेकदा **एका location** आणि **एका logical AWS endpoint** पुरता असतो. Location किंवा तो endpoint गेला तर LAG जातो.
- **LAG bandwidth देतो. Multi-site resiliency देत नाही.**
- Region मागे **10 LAG**. वाढ मागता येते.
- प्रत्येक LAG member Region च्या connection कोट्यात मोजला जातो.
- LAG तयार झाल्यावर **VIF LAG वर** तयार होतो, खालच्या एका connection वर नाही.
- VIF कोटा LAG मागे **51**, वाढत नाही. त्यात transit VIF ही मोजले जातात. Transit ची स्वतंत्र मर्यादा dedicated connection प्रमाणे **4**.
- Minimum links हा attribute डीफॉल्ट **0**. तुम्ही वाढवू शकता. चालू links त्या संख्येखाली गेले तर **संपूर्ण LAG** खाली जातो. उरलेल्या links वर overload टाळण्यासाठी हे वापरतात.
- Public, private आणि transit VIF LAG वर चालतात.
- SLA गणनेत संपूर्ण LAG **एक** Connection आहे.

MACsec असलेले connections LAG मध्ये घालताना जुन्या keys निघून LAG ची key लागते. एका वेळी LAG वर एकच MACsec key वापरायची. अनेक keys फक्त rotation साठी.

---

## 9. MACsec

**MACsec** IEEE **802.1AE** आहे. ते customer device आणि DX PoP यांच्यातील Ethernet link वर confidentiality, integrity आणि origin authenticity देते. हा hop **point-to-point** आहे. Instance पर्यंत end-to-end encryption नाही.

### कुठे चालते

- **10, 100 आणि 400 Gbps dedicated** connections, आणि असे LAG.
- फक्त **निवडक PoP**.
- **1 Gbps dedicated वर नाही.**
- **कोणत्याही hosted connection वर नाही.**
- Partner आपल्या स्वतःच्या interconnect वर MACsec वापरू शकतो. तो ग्राहकाच्या hosted connection वर येत नाही.
- MACsec साठी **वेळेचे charge नाही.**

Cipher, docs प्रमाणे:

| Speed | Cipher |
| --- | --- |
| 10 Gbps | **GCM-AES-256** आणि **GCM-AES-XPN-256** |
| 100 Gbps आणि 400 Gbps | **GCM-AES-XPN-256** |

फक्त **256-bit** keys. 100G आणि 400G ला **XPN** आवश्यक आहे, कारण 32-bit packet number लवकर संपतो. **SCI चालू हवा** आणि तो बंद करता येत नाही. **Dot1q-in-clear support नाही.** VLAN tag encrypted payload च्या बाहेर काढता येत नाही.

ज्या device वर Layer 2 संपतो, त्याला MACsec हवा. मधला last-mile पारदर्शक हवा, किंवा तो device स्वतः MACsec terminate करतो. Partner चा last-mile support नसेल तर dedicated port MACsec-capable असूनही उपयोग नाही.

### कळा

- **CKN/CAK** तुम्ही तयार करता. Hexadecimal, **नेमके 64 characters**. AES-256.
- फक्त **static CAK**. Dynamic CAK नाही.
- **SAK** CKN/CAK पासून तयार होतो. तो pre-shared नाही. वेळ, traffic volume, किंवा नव्या session नुसार आपोआप फिरतो.
- Keychain मध्ये **तीन** CKN/CAK जोड्या. नवी key `associate-mac-sec-key` ने द्या आणि router वर तीच लावा. जुनी key fallback म्हणून राहते, त्यामुळे rotation मध्ये link तुटू नये.
- AWS या keys Secrets Manager मध्ये ठेवतो, AWS managed key ने. Secret delete schedule केल्यास शेवटची key निघू शकते.

Modes:

- **`should_encrypt`**: MACsec न झाल्यास unencrypted traffic चालू राहतो. नव्या MACsec connection चे default हे आहे, म्हणजे पहिल्या सेटअपमध्ये link पूर्ण बंद होऊ नये.
- **`must_encrypt`**: MACsec शिवाय traffic नाही. जास्त सुरक्षित. Negotiation अयशस्वी झाल्यास उपलब्धता जाते.

`must_encrypt` असताना शेवटची CKN निघाली तर AWS mode `should_encrypt` करतो, जेणेकरून अचानक सर्व packets गिरवले जाऊ नयेत.

### SiteLink सोबत

SiteLink स्वतःचा cipher नाही. तो VIF सुविधा आहे. खालचा **dedicated** port आणि PoP MACsec support करत असतील तर त्या link वरील SiteLink traffic देखील PoP पर्यंत MACsec ने झाकलेला असतो. Hosted connection वर MACsec नसल्याने hosted SiteLink त्या हॉपवर MACsec ने encrypt होत नाही.

### अजून encryption हवे असेल तर

- PoP पर्यंत: MACsec.
- VGW किंवा TGW पर्यंत Layer 3: IPsec VPN over public VIF, किंवा private IP VPN over Direct Connect.
- Application: TLS.
- AWS backbone वरील physical-layer encryption customer key चे बदली धरू नका.

---

## 10. SiteLink

**SiteLink** दोन किंवा अधिक DX PoP जोडतो, जेणेकरून साइट्स एकमेकींना AWS backbone वरून बोलू शकतील. Traffic **Region किंवा VPC मधून hairpin होत नाही.**

- Private आणि transit VIF वर, खालील तक्त्याप्रमाणे.
- Hosted आणि dedicated दोन्हीवर.
- **Public VIF वर नाही.**
- **थेट VGW ला लागलेल्या private VIF वर नाही.**
- **AWS GovCloud (US) आणि China Regions मध्ये नाही.**
- तोच route अनेक VIF वरून advertise केला तर SiteLink काम करत नाही.
- VPC connectivity चे बदली नाही. साइट ते साइट आहे.
- Jumbo: VIF प्रकारानुसार 8500 किंवा 9001.

| VIF | SiteLink |
| --- | --- |
| Transit | Supported |
| Private, DX Gateway ला, VGW associated असो वा नसो | Supported |
| Private, थेट VGW | **नाही** |
| Public | **नाही** |

SiteLink चालू असताना Region कडून on-prem कडे जाणारा traffic कमी AS path पसंत करतो, associated Region पेक्षाही. SiteLink बंद असताना associated Region चा location पसंत केला जातो, जरी दुसरीकडे AS path लहान असेल.

किंमत विभाग 12 मध्ये. थोडक्यात: enabled VIF मागे तासाचे charge, अधिक PoP ते PoP data transfer. Flat-rate tier मध्ये SiteLink येत नाही.

---

## 11. कोटा तक्ता

स्रोत: Direct Connect quotas page. “वाढत नाही” तिथे लिहिले आहे तिथे पाठ करा.

| कोटा | मूल्य | वाढ |
| --- | --- | --- |
| Private किंवा public VIF per dedicated connection | **50** | नाही |
| Transit VIF per dedicated connection | **4** | SA किंवा TAM |
| एकूण VIF per dedicated connection | **51** | **नाही** |
| VIF per hosted connection | **1** (private, public किंवा transit) | नाही |
| Active DX connections per location per Region per account | **10** | SA किंवा TAM |
| VIF per LAG | **51** | नाही |
| Dedicated connections per LAG | **4** जर speed 100G पेक्षा कमी; **2** जर 100G. 400G ही याच **2** च्या नियमात | — |
| LAG per Region | **10** | विनंती |
| Rate limiters per dedicated connection | **10** | SA किंवा TAM |
| On-prem → AWS prefixes, private/transit, प्रति family | **100** default, **1,000** पर्यंत | Prefix controls / SA / TAM |
| Public VIF prefixes | **1,000** | नाही |
| TGW → on-prem, transit VIF | **200** IPv4+IPv6 मिळून | विनंती |
| Cloud WAN → on-prem | **5,000** | विनंती |
| DX Gateway per account | **200** | विनंती |
| VGW per DX Gateway | **20** | नाही |
| TGW per DX Gateway | **6** | नाही |
| Private किंवा transit VIF per DX Gateway | **30** | नाही |
| DX Gateway per TGW | **20** | नाही |
| VIF per VGW | मर्यादा नाही | — |
| SiteLink prefixes | IPv4 1,000 आणि IPv6 1,000 पर्यंत, prefix controls | SA किंवा TAM |

51 ची मर्यादा अशी तयार झाली: आधी 50 private किंवा public. Transit आल्यावर एक जागा वाढली, एकूण 51. आता transit 4 असू शकतात, पण ते 51 च्या आत मोजले जातात. **51 वाढवता येत नाही.**

Rate limiter dedicated connection वरील VIF ची bandwidth मर्यादित करतो. Hosted connection वर क्षमता पोलिसिंग आधीच असते.

Public VIF चे load balancing हवे असेल तर त्या सर्व VIF **एकाच Region** मध्ये हव्यात.

---

## 12. किंमत, संकल्पना

डॉलरचे port दर इथे नाहीत. ते location आणि capacity नुसार बदलतात. Pricing page पाहा.

दोन मुख्य बाबी: **port-hour** आणि **data transfer out**.

- Port-hour त्या तासांसाठी, जेव्हा port तुमच्या वापरासाठी provisioned असतो. Traffic नसला तरी लागू.
- **अर्धा तासही पूर्ण तास** म्हणून बिल होतो. FAQ चा नियम.
- **Connection owner** port-hour देतो.
- **Data transfer in** Direct Connect वर charge होत नाही. Pay-as-you-go पान: सर्व locations मध्ये **$0.00 per GB**.
- Data transfer out प्रति GB, Region आणि DX location नुसार.
- VIF तयार केल्याबद्दल वेगळे port charge नाही.
- MACsec साठी वेगळे charge नाही.
- Multi-account DX Gateway साठी वेगळे Direct Connect charge नाही.

### कोण DTO देतो

FAQ आणि User Guide:

- **Private VIF:** ज्या AWS account च्या resources नी data बाहेर पाठवला, तो account. DX Gateway असो वा नसो.
- **Transit VIF:** ज्या VPC चा owner, तो account. त्या VPC त्या TGW ला लागलेल्या असतात, तो TGW त्या DX Gateway ला, आणि तो DX Gateway त्या transit VIF ला. TGW data processing आणि attachment वेगळे लागतात.
- **Public VIF:** publicly addressable resource चा DTO, resource owner कडे, Direct Connect दराने, जेव्हा गंतव्य public prefixes त्याच payer account च्या मालकीचे असतात आणि public VIF वर advertise होतात.
- Blog नुसार बारकावे: TGW मागून शेवटचा resource जो data DX कडे पाठवतो, त्याचा owner DTO देतो. बाहेरून VPN ने TGW मध्ये येऊन पुन्हा बाहेर जाणारा traffic असल्यास DTO TGW owner कडे जाऊ शकतो. नेमका प्रवाह सांगितल्याशिवाय “नेहमी resource owner” म्हणू नका.

### Hosted कोण बिल करतो

- FAQ: hosted connection **accept** केल्यानंतर port-hour सुरू. जोपर्यंत connection provisioned आहे तोपर्यंत चालू. थांबवायचे असेल तर Partner कडून cancel. बिलावर गट `HCPortUsage:50M` सारखा दिसतो.
- Pricing guide हेही म्हणते: hosted port-hour च्या व्यवस्थेत **Partner** असू शकतो. Partner ला विचारा.
- **Last-mile, cross-connect आणि colocation** AWS च्या port-hour मध्ये येत नाहीत. ते provider किंवा facility बिल करतात.

दोन्ही वाक्ये खरी ठेवा. AWS bill वर hosted port-hour येऊ शकतो, आणि Partner वेगळा last-mile बिल करतो. कोणाला काय दिसते ते कराराने बदलेल.

### Flat-rate

Dedicated connections साठी pay-as-you-go सोबत **flat-rate** आहे. ठरावीक तासाचा दर, bandwidth आणि geographic tier नुसार. निवडलेल्या tier मधील Regions साठी प्रति GB DTO त्या दरात येते. Tier बाहेरच्या Region चा DTO मानक दराने. Data transfer in येथेही charge नाही. तास पुढच्या पूर्ण तासापर्यंत.round होतात.

Flat-rate मध्ये private, public आणि transit traffic येते. **SiteLink येत नाही.** Pricing guide नुसार port-pair मधील redundant दुसरा port काही flat-rate रचनांमध्ये आणखी port charge शिवाय असू शकतो. चालू अटी pricing page वर पाहा.

### SiteLink किंमत

दोन बाबी, अधिकृत pricing page:

- **SiteLink hours:** ज्या VIF वर SiteLink enabled आहे, त्या मागे तासाचे charge. Traffic नसला तरी. पानावर प्रकाशित दर **$0.50 per hour per VIF**, सर्व locations आणि speeds साठी. दर बदलू शकतो.
- **SiteLink data transfer:** PoP ते PoP, प्रति GB, source आणि destination नुसार. ही matrix symmetric आहे. आकडे इथे चढवले नाहीत.

### DX की VPN स्वस्त

- Traffic कमी, तास कमी, किंवा कधीतरी जोडणी हवी: **VPN** स्वस्त पडू शकते. Port-hour नसतो. Internet egress आणि VPN तास लागतात.
- Traffic जास्त, दर तासाला स्थिर, latency स्थिर हवी: **DX** योग्य. Idle port देखील पैसे घेतो. खरेदी केलेली capacity वापरली नाही तर port-hour वाया जातो.
- DX चा data transfer out अनेकदा internet egress पेक्षा कमी आणि अधिक स्थिर असतो. हा तुलनात्मक कल आहे. बिल काढायचे असेल तर चालू pricing page वापरा.

---

## 13. Multi-account आणि hybrid नमुने

### एक connection, अनेक खाती

Networking account connection ठेवतो आणि port-hour देतो. VIF दुसऱ्या account च्या मालकीची असते. तो account VIF accept करतो. Private VIF त्या account च्या VGW किंवा DX Gateway कडे जाते. Public VIF चा DTO resource owner कडे जातो.

### DX Gateway networking account मध्ये

- Spoke account VGW किंवा TGW ठेवतो.
- Association **proposal** पाठवतो.
- Networking account accept करतो.
- DX Gateway owner allowed prefixes ठेवू शकतो किंवा बदलू शकतो.
- TGW association cross-account आहे. हा सध्याचा मॉडेल आहे.
- Transit VIF + DX Gateway + TGW हा अनेक VPC आणि अनेक accounts साठी मानक enterprise नमुना आहे.
- एक VPC, एक Region, साधे routing: private VIF थेट VGW.

### S3 कसे गाठावे

| मार्ग | काय होते |
| --- | --- |
| **Public VIF** | S3 चा public endpoint, Amazon public prefixes, public IP |
| **Private VIF एकटा** | S3 public endpoint ला जात नाही. Private VIF VPC CIDR साठी आहे |
| **S3 gateway endpoint** | VPC च्या आत route-table endpoint. **On-prem, VPN, peering, TGW किंवा Direct Connect कडून gateway endpoint वापरता येत नाही** |
| **S3 interface endpoint (PrivateLink)** | VPC मध्ये private IP. On-prem ते IP private किंवा transit VIF ने गाठू शकतो, routes, security groups आणि endpoint policy जुळले तर |

“Private VIF अधिक VPC endpoint” असे अर्धवट वाक्य बोलू नका. **Interface endpoint** होय. **Gateway endpoint** on-prem साठी नाही.

### Multicast

Direct Connect चा सामान्य डेटा मार्ग unicast BGP आहे. **Transit Gateway multicast Direct Connect, Site-to-Site VPN, peering attachment आणि TGW Connect वर support नाही.** Multicast फक्त VPC attachments मधील subnets मध्ये. DX वर multicast चालेल असे गृहित धरू नका.

### आणखी नमुने

- अनेक Regions, VGW: एक private VIF, एक DX Gateway, वीस VGW पर्यंत.
- Cloud WAN: core network DX Gateway ला associate. Prefixes 5,000 पर्यंत. सर्व transit VIF ला तेच prefixes.
- Local Zone: VGW मार्गे DX Gateway, docs मध्ये वेगळा नमुना आहे.
- SD-WAN appliance: TGW Connect attachment चा transport VPC किंवा Direct Connect attachment असू शकतो. Connect हा GRE आणि BGP आहे. तो SiteLink किंवा MACsec चे बदली नाही.

---

## 14. Troubleshooting

### Connection states

| State | अर्थ |
| --- | --- |
| `requested` | Dedicated connection ची सुरुवात. LOA ग्राहकाकडे जाईपर्यंत |
| `ordering` | Hosted connection ची सुरुवात. मालक confirm किंवा नाकारेपर्यंत |
| `pending` | मंजूर, initialize होत आहे |
| `available` | Link वर आहे, वापरासाठी तयार |
| `down` | Link खाली |
| `deleting` / `deleted` | काढले जात आहे / काढले |
| `rejected` | `ordering` मधील hosted connection ग्राहकाने काढली |
| `unknown` | State उपलब्ध नाही |

### VIF states

| State | अर्थ |
| --- | --- |
| `confirming` | दुसरा account accept करणार |
| `verifying` | **फक्त public VIF.** Prefix मालकी तपासली जात आहे |
| `pending` | तयार झाली, traffic आधी |
| `available` | Traffic जाऊ शकतो |
| `down` | **BGP down** |
| `testing` | Failover test चालू आहे |
| `deleting` / `deleted` / `rejected` / `unknown` | काढणे, नाकारणे, किंवा अज्ञात |

### VIF down

क्रमाने पाहा:

1. Connection खरोखर `available` आहे का, की फक्त VIF खाली आहे.
2. **VLAN** दोन्ही बाजू समान आहे का. Hosted वर Partner चा VLAN वापरला का. तयार केल्यानंतर VLAN बदलत नाही.
3. Peer IP आणि mask दोन्ही बाजू जुळतात का. `/31` असेल तर दोन पत्ते शेजारी हवेत.
4. **MD5** जुळतो का.
5. दोन्ही बाजू **समान ASN** तर नाही ना.
6. Firewall **TCP 179** अडवत आहे का.
7. Peer IP पर्यंत route आहे का. BGP TTL **1** आहे. Multihop चालणार नाही. Peer IP दुसऱ्या subnet मागे असेल तर session येणार नाही.
8. 802.1Q मधल्या device ने खाल्ले तर नाही ना.
9. Dedicated port वर auto-negotiation त्या endpoint ला शोभतो का.

### BGP established, traffic नाही

- Route advertise झाला का. `VirtualInterfaceBgpPrefixesAccepted` शून्य असेल तर AWS ला on-prem prefixes मिळाले नाहीत.
- TGW association चे **allowed prefixes** रिकामे किंवा चुकीचे तर on-prem ला VPC दिसणार नाही.
- VPC route table VGW किंवा TGW कडे बघते का.
- **Security group आणि NACL** अजून लागू आहेत. Direct Connect त्यांना वगळत नाही.
- परतीचा मार्ग दुसरा आहे. Stateful firewall asymmetric traffic टाकतो.
- VPN वर अधिक specific prefix DX ला हरवतो.
- Public VIF अजून `verifying` असेल तर prefixes approve झालेले नाहीत.
- Hosted connection accept केलेला नसेल तर VIF तयार होत नाही.
- Prefix quota ओलांडली तर session idle आणि DOWN. “अर्धा traffic” नाही.

### Jumbo

मोठे packets गिरतात, ping छोटे असल्याने चालू दिसते. दोन्ही बाजू MTU जुळवा. Private 9001, transit 8500, public 1500. End-to-end कुठेही 1500 असेल तर मोठे frames तिथे पडतात. DF bit असलेला ping, हळूहळू वाढवलेला size, हा चाचणी आहे. Production ping flood करू नका.

### CloudWatch

डीफॉल्ट interval 5 मिनिटे. किमान काही metrics 1 मिनिटापर्यंत खाली येतात. नवीन connection किंवा device reboot नंतर bitrate काही वेळ दिसणार नाही.

Connection:

- `ConnectionState` — 1 up, 0 down. Dedicated आणि hosted.
- `ConnectionBpsEgress`, `ConnectionBpsIngress`
- `ConnectionPpsEgress`, `ConnectionPpsIngress`
- `ConnectionErrorCount` — MAC errors, त्यात CRC. जुने `ConnectionCRCErrorCount` वापरू नका. Sum statistic.
- `ConnectionLightLevelTx`, `ConnectionLightLevelRx` — **dBm**. Dimension `OpticalLaneNumber`. कमकुवत Rx म्हणजे fiber, optic, किंवा cross-connect.
- `ConnectionEncryptionState` — MACsec सारखी encryption up की down. LAG वर 1 म्हणजे सर्व members up.
- `ConnectionDiscardsPpsEgress` — buffer किंवा congestion मुळे टाकलेले packets.

VIF:

- `VirtualInterfaceBpsEgress`, `VirtualInterfaceBpsIngress`
- `VirtualInterfacePpsEgress`, `VirtualInterfacePpsIngress`
- `VirtualInterfaceBgpStatus` — 1 up, 0 down. Dimension `IpAddressFamily`: `ipv4` किंवा `ipv6`.
- `VirtualInterfaceBgpPrefixesAccepted`
- `VirtualInterfaceBgpPrefixesAdvertised`

Physical शंका असेल तर आधी light level आणि `ConnectionErrorCount`. BGP शंका असेल तर `VirtualInterfaceBgpStatus` आणि prefix counts. Capacity शंका असेल तर Bps विरुद्ध खरेदी केलेली speed. Hosted वर पोलिसिंगमुळे सरासरी Bps खरेदीखाली दिसू शकते, जरी bursts drop होत असले.

Console मधील failover test, किंवा `StartBgpFailoverTest`, BGP मुद्दाम खाली आणतो. चाचणीनंतर session परत आला का ते पाहा.

---

## 15. सुरक्षा

- Direct Connect **खाजगी मार्ग** आहे. **डीफॉल्ट encryption नाही.**
- Physical hop: MACsec, जिथे support आहे.
- Layer 3: IPsec over DX.
- Application: TLS.
- VPC traffic internet ला नको असेल तर **private किंवा transit VIF**. Public VIF internet नाही, पण Amazon public prefixes आणि public services आहेत.
- Public VIF वर फक्त स्वतःचे prefixes. AWS मालकी तपासतो. तरी router वर filter ठेवा. AWS कडून आलेले prefixes `NO_EXPORT` ने चिन्हांकित असतात. त्यांना internet कडे जाऊ देऊ नका.
- IAM ने कोण connection, VIF, LAG, DX Gateway आणि MACsec key बनवू शकतो ते मर्यादित करा.
- **LOA-CFA संवेदनशील कागद** आहे. ज्याच्याकडे पत्र आहे, तो facility मध्ये तुमच्या port पर्यंत cross-connect मागू शकतो. कोण download करतो ते नियंत्रित करा.
- MACsec key साठी console किंवा CLI वापरताना AWS **TLS 1.3** आणि **ML-KEM** सारखा post-quantum key exchange सुचवते.
- `must_encrypt` उपलब्धतेचा निर्णय आहे. तो सुरक्षा निर्णयही आहे. Negotiation अयशस्वी झाल्यास traffic थांबतो.
- Security groups, NACLs, आणि TGW route tables अजून लागू आहेत.

---

## 16. बांधणीचा क्रम

चर्चेत हा क्रम न अडखळता सांगा.

### Dedicated

1. Console मध्ये connection तयार करा. Production असेल तर Resiliency Toolkit. एकल जोडणी Classic.
2. मंजुरीनंतर **LOA-CFA** घ्या. ओळखीचा Partner आणि जुने account असेल तर LOA लगेच मिळू शकते.
3. Facility किंवा network provider cross-connect बसवतो. Provider DX Partner असणे dedicated साठी आवश्यक नाही.
4. Connection `available` होईपर्यंत वाट पाहा. `down` असेल तर fiber, optic, auto-negotiation.
5. जास्त bandwidth एका endpoint वर हवी असेल तर **LAG**. वेगळ्या locations ची resiliency हवी असेल तर LAG ऐवजी दुसरी connection.
6. Private, public, किंवा transit **VIF**. VLAN, ASN, MD5, peer IP.
7. Customer router वर BGP. शक्यतो BFD. Graceful restart सोबत BFD ठेवू नका.
8. Private असेल तर त्याच Region चा VGW, **किंवा** DX Gateway आणि नंतर VGW associations.
9. Transit असेल तर VIF DX Gateway ला, TGW associate करा, **allowed prefixes** लिहा, VPC route table TGW कडे वळवा.
10. Public असेल तर prefixes approve होईपर्यंत `verifying`.
11. दुसरा location, दुसरा router, दुसरा last-mile.
12. Failover test. एक BGP खाली आणा. Traffic दुसरीकडे गेला का ते पाहा.

### Hosted

1. **DX Partner** कडे capacity मागा. Console मधून ही विनंती होत नाही.
2. Account मध्ये hosted connection **accept** करा. तोपर्यंत `ordering`.
3. त्या एका connection वर **एक** VIF तयार करा: private, public, किंवा transit.
4. Router वर BGP.
5. Gateway जोडणी वरीलप्रमाणे.
6. last-mile चा बिल आणि SLA Partner कडे आहे हे वेगळे लक्षात ठेवा. Dedicated SLA hosted ला लागू होत नाही.

---

## 17. निर्णय तक्ता

| प्रश्न | उत्तर |
| --- | --- |
| Dedicated की hosted | Port पूर्णपणे तुमचा हवा, अनेक VIF, MACsec, LAG, 100G किंवा 400G, SLA: **dedicated**. 50 Mbps ते 25 Gbps, Partner port, एक VIF, लवकर सुरुवात: **hosted** |
| Private की public की transit | एक किंवा काही VPC, private IP: **private**. S3/API public IP ने: **public**. अनेक VPC, अनेक accounts, TGW किंवा Cloud WAN: **transit** |
| DX Gateway की थेट VGW | एक VPC, त्याच Region: थेट **VGW** पुरे. अनेक Regions किंवा अनेक VGW: **DX Gateway**. TGW असेल तर DX Gateway सक्तीचा आहे |
| LAG की दुसरे location | एका ठिकाणी जास्त Gbps: **LAG**. Location failure सहन करायचे असेल: **दुसरे location**. 99.99 साठी LAG एक connection मोजला जातो |
| MACsec की VPN | फक्त PoP पर्यंतचा hop, 10G किंवा अधिक dedicated: **MACsec**. Instance किंवा VGW/TGW पर्यंत IPsec, किंवा 1G/hosted: **VPN**. Application साठी दोन्ही असूनही **TLS** |
| SiteLink की VPC peering की TGW peering | साइट ते साइट, Region टाळून: **SiteLink**. दोन VPC: **peering** किंवा **TGW**. On-prem अनेक VPC: transit VIF. DX Gateway VPC peering चे काम करत नाही |
| DX की VPN की Interconnect last mile | स्वतःचे BGP आणि VIF हवे: **DX**. कमी traffic: **VPN**. Partner last-mile, चार links आणि port SLA हाताने नको: **Interconnect last mile**, उपलब्ध असेल तिथे |
| S3 | Public endpoint: **public VIF**. खाजगी IP: **interface endpoint** अधिक private/transit VIF. Gateway endpoint on-prem साठी नाही |

---

## शेवटची 10 मिनिटे — memory sheet

हा एक पानांचा सारांश आहे. 50 व्या मिनिटापासून फक्त हे आणि खालील प्रश्न.

**मार्ग:** router → cross-connect → DX endpoint → backbone → Region. LOA-CFA शिवाय facility cross-connect करत नाही.

**Speeds:** Dedicated **1 / 10 / 100 / 400**. Hosted **50, 100, 200, 300, 400, 500 Mbps, 1, 2, 5, 10, 25 Gbps**. Optics: 1000BASE-LX 1310, 10GBASE-LR 1310, 100GBASE-LR4, 400GBASE-LR4.

**VIF:** Private = VPC private IP. Public = AWS public IP, customer public peering IP, jumbo नाही. Transit = फक्त DX Gateway, मग TGW. Hosted connection = **1 VIF**. VLAN **1–4094**, नंतर बदल नाही.

**DX Gateway:** Global. China नाही. GovCloud होय. VGW **20**. TGW **6**. VIF **30**. Account मागे **200**. VPC-to-VPC router **नाही**. Supernet overlap हा अपवाद, डिझाइन नाही.

**BGP:** eBGP, TTL 1, MD5 सक्तीचा. समान ASN दोन्ही बाजू नाही. Private/transit inbound **100/100**, कमाल **1000/1000**, ओलांडल्यास session DOWN. Public **1000** कडक. TGW कडून **200** मिळून. Cloud WAN **5000**.

**Communities, पाठ:**

- AWS कडे scope: **9100** local, **9200** continent, **9300** global. रिकामे म्हणजे global.
- AWS कडून: **8100** same Region, **8200** same continent, tag नाही म्हणजे इतर खंड.
- **NO_EXPORT** वेगळे. ते 9100 नाही.
- Local pref: **7100** low, **7200** medium, **7300** high.

**SLA:** 99.99 = चार dedicated, दोन locations, प्रत्येकी दोन, वेगळे devices, दोन AZ, Enterprise Support, Well-Architected Review. 99.9 = दोन locations, किमान दोन connections, Enterprise Support. 95 = एक connection. Hosted SLA बाहेर. LAG = एक connection.

**LAG:** एक endpoint, MLAG नाही. 100G/400G कमाल 2. त्याखाली कमाल 4. Location failure सहन करत नाही.

**MACsec:** 10/100/400 dedicated, निवडक PoP. 1G नाही. Hosted नाही. 10G: GCM-AES-256 आणि GCM-AES-XPN-256. 100/400: फक्त GCM-AES-XPN-256. SCI सक्ती. Charge नाही. End-to-end नाही.

**MTU:** Public 1500. Private 9001. Transit 8500.

**BFD:** 300 ms, multiplier 3. GR सोबत नको.

**SiteLink:** Private DX Gateway VIF आणि transit. Public नाही. थेट VGW नाही. GovCloud आणि China नाही. तास अधिक data transfer.

**पैसे:** Owner port-hour. अर्धा तास पूर्ण तास. DTO resource किंवा VPC owner. Data in ला charge नाही. VIF ची वेगळी port किंमत नाही.

---

## 18. जलद प्रश्न आणि अचूक उत्तरे

प्रत्येक उत्तर आधी होय किंवा नाही, मग एक वाक्य.

1. **एका hosted connection वर तीन VIF काढता येतील का?** नाही. फक्त एक. तो private, public किंवा transit असू शकतो.

2. **Direct Connect traffic डीफॉल्टने encrypt करतो का?** नाही. MACsec, IPsec, किंवा TLS शिवाय customer encryption नाही. Backbone वरील AWS physical-layer encryption वेगळी गोष्ट आहे.

3. **Private VIF थेट S3 public endpoint गाठतो का?** नाही. S3 public साठी public VIF. खाजगी IP साठी S3 interface endpoint आणि private routing. Gateway endpoint on-prem वरून चालत नाही.

4. **DX Gateway दोन VPC एकमेकांशी जोडतो का?** सामान्य डिझाइनमध्ये नाही. फक्त on-prem ते VPC. Overlapping supernet एकाच VIF वर असेल तर docs सांगते की VPC एकमेकांशी DX endpoint मार्गे बोलू शकतात. तो अपवाद टाळा.

5. **LAG multi-site resilient आहे का?** नाही. एक endpoint, अनेकदा एक location. Location गेला तर LAG जातो.

6. **1 Gbps वर MACsec आहे का?** नाही.

7. **Hosted connection वर MACsec आहे का?** नाही.

8. **दोन बाजू समान ASN असतील तर?** BGP स्थापित होत नाही.

9. **एका connection ने 99.99% मिळेल का?** नाही. 99.99 ला किमान चार dedicated connections, दोन locations, वेगळे devices, आणखी Support आणि review अटी.

10. **VLAN नंतर बदलता येईल का?** नाही.

11. **Public VIF वर jumbo आहे का?** नाही. MTU 1500.

12. **Private jumbo किती?** 9001. Transit jumbo 8500.

13. **Transit VIF थेट TGW ला लागते का?** नाही. आधी DX Gateway. मग TGW association आणि allowed prefixes.

14. **Public peering साठी EIP चालेल का?** नाही. EIP किंवा Amazon pool BYOIP नको. स्वतःचे public IP.

15. **Port-hour कोण देतो?** Connection owner. Hosted accept केल्यानंतर port-hour सुरू होतो. Last-mile Partner वेगळा बिल करू शकतो.

16. **Data transfer in ला charge आहे का?** नाही. अधिकृत पान $0.00 per GB म्हणते.

17. **NO_EXPORT म्हणजे 7224:9100 का?** नाही. 9100 local Region scope आहे, customer ते AWS. NO_EXPORT well-known community आहे, AWS ने दिलेल्या public prefixes वर.

18. **Continent scope कोणती community?** AWS कडे prefixes देताना **7224:9200**. AWS कडून येताना त्याच खंडाची tag **7224:8200**.

19. **Global scope?** **7224:9300**. Community नसेल तरही public prefix default global जातो.

20. **High local preference?** **7224:7300**. Low **7224:7100**. Medium **7224:7200**.

21. **Private VIF वर 150 prefixes दिल्यास?** Allocation 100 असेल तर BGP idle आणि DOWN. आधी prefix control 1,000 पर्यंत वाढवा. वाढवल्याशिवाय 101 वा prefix session पाडतो.

22. **Public VIF prefix मर्यादा वाढते का?** नाही. 1,000 कडक.

23. **एका VIF वर दोन IPv4 BGP sessions?** नाही. IPv4 एक, IPv6 एक, किंवा दोन्ही.

24. **BGP TTL?** 1. Multihop नाही.

25. **BFD चे documented minimum?** Interval 300 ms, multiplier 3. दोन्ही बाजू त्यावर असतील तर detection सुमारे 900 ms. Graceful restart सोबत ठेवू नका.

26. **Dedicated साठी DX Partner सक्तीचा आहे का?** नाही. Fiber कोणताही network provider देऊ शकतो. Hosted मात्र फक्त DX Partner.

27. **400 Gbps LAG मध्ये येतो का?** होय. 100G प्रमाणे जास्तीत जास्त दोन members. समान endpoint.

28. **51 VIF वाढवता येतील का?** नाही. Dedicated connection आणि LAG दोन्हीवर.

29. **VGW per DX Gateway किती?** 20. वाढत नाही. TGW per DX Gateway 6. वाढत नाही. VIF per DX Gateway 30. वाढत नाही.

30. **SiteLink public VIF वर आहे का?** नाही. थेट VGW ला लागलेल्या private VIF वरही नाही.

31. **China VPC commercial DX Gateway वर येईल का?** नाही. GovCloud येऊ शकतो. SiteLink GovCloud आणि China मध्ये नाही.

32. **Hosted connection ला 99.99 SLA लागू आहे का?** नाही. Partner hosted connection आणि hosted virtual interface SLA बाहेर आहेत.

33. **LAG चे चार ports SLA मध्ये चार connections मोजले जातात का?** नाही. LAG एक Connection आहे.

34. **Multicast DX वरून TGW multicast domain मध्ये जाईल का?** नाही. TGW multicast Direct Connect वर support नाही.

35. **Security group DX मुळे बाजूला होते का?** नाही. SG, NACL आणि route table अजून लागू आहेत.

36. **दोन connections वर तोच customer ASN चालेल का?** होय. AWS ASN देखील वेगवेगळ्या connections वर तोच असू शकतो. एका VIF च्या दोन बाजू समान असू शकत नाहीत.

37. **DX Gateway चा default ASN?** 64512. VGW चा default ही 64512. तयार केल्यानंतर VGW ASN बदलत नाही.

38. **Public VIF `verifying` मध्ये का अडकते?** AWS prefix ची मालकी तपासत आहे. Approve शिवाय पुढे नाही.

39. **Connection `available` आणि तरी traffic नाही?** VIF, BGP, allowed prefixes, route table, SG/NACL, किंवा अधिक specific VPN route तपासा. Physical link आला म्हणजे VPC route आला असे नाही.

40. **कमकुवत fiber कसे दिसेल?** `ConnectionLightLevelRx` आणि `ConnectionLightLevelTx`, एकक dBm, dimension `OpticalLaneNumber`. सोबत `ConnectionErrorCount`.

---

## अधिकृत स्रोत

- [Direct Connect quotas](https://docs.aws.amazon.com/directconnect/latest/UserGuide/limits.html)
- [Virtual interfaces](https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html)
- [Routing policies आणि BGP communities](https://docs.aws.amazon.com/directconnect/latest/UserGuide/routing-and-bgp.html)
- [BGP settings](https://docs.aws.amazon.com/directconnect/latest/UserGuide/bgp-specific-settings.html)
- [Direct Connect gateways](https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-gateways-intro.html)
- [Allowed prefixes](https://docs.aws.amazon.com/directconnect/latest/UserGuide/allowed-to-prefixes.html)
- [Resiliency Toolkit](https://docs.aws.amazon.com/directconnect/latest/UserGuide/resiliency_toolkit.html)
- [SLA](https://aws.amazon.com/directconnect/sla/)
- [MACsec](https://docs.aws.amazon.com/directconnect/latest/UserGuide/MACsec.html)
- [LAG](https://docs.aws.amazon.com/directconnect/latest/UserGuide/lags.html)
- [CloudWatch metrics](https://docs.aws.amazon.com/directconnect/latest/UserGuide/monitoring-cloudwatch.html)
- [FAQ](https://aws.amazon.com/directconnect/faqs/)
- [Pricing](https://aws.amazon.com/directconnect/pricing/)
- [Connection states, API](https://docs.aws.amazon.com/directconnect/latest/APIReference/API_Connection.html)
- [Virtual interface states, API](https://docs.aws.amazon.com/directconnect/latest/APIReference/API_VirtualInterface.html)

Pricing pages वरील डॉलर दर आणि Interconnect चे Region तसेच provider यादी बदलतात. चर्चा आधी त्या पानांवर डोळा फिरवा. या नोट्समधील कोटा, communities, cipher, SLA अटी आणि state नावे वरच्या दस्तऐवजांशी जुळवून लिहिली आहेत.
