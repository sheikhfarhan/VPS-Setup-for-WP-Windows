## Part 4 - Cloudflare Setup

#### :arrow_right_hook: Point Nameservers to Cloudflare

Go to Domain Registrar and add Cloudflare's given nameservers

![image](https://drive.google.com/uc?export=view&id=14pLjdHOKozSED2eVMWjopYNb69k1Bm9b)

#### :arrow_right_hook: Adding A & CNAME records for domain

![image](https://drive.google.com/uc?export=view&id=153-rR06znAnvLD67EgRvgoNM1MkaSM2w)

> [!NOTE]
> CloudPanel's log-in page will be clp.domain.com\
> Do not proxy it through Cloudflare yet!

#### :arrow_right_hook: Switch to Full (Strict) mode

SSL/TLS -> Overview -> Configure

![image](https://drive.google.com/uc?export=view&id=1-jeIhfHYPGM-BvZVWfdL2Ba89uNhXZlh)

#### :arrow_right_hook: Create CF Certificate

SSL/TLS -> Origin Server -> Create

![image](https://drive.google.com/uc?export=view&id=151xiP1wt7WIU5KUP-9MVJPDYzMtG4556)

Copy the two sets of keys somewhere. We will need them later. Once copied, click OK.

