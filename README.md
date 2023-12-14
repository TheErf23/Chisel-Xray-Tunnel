
# <p align="center">Chisel معرفی و آموزش تانل</p>
  
همینطور که میدونید، اگه بخوایم از CDN کلادفلر یا مشابه اون استفاده کنیم و سرورمون رو پشت CDN مخفی کنیم، فقط می‌تونیم از ترنزمیشن های Ws و gRPC استفاده کنیم. دلیلشم اینه که در پلن رایگان کلادفلر، انتقال ترافیک‌های TCP و UDP ممکن نیست و برای این کار نیاز به تهیه پلن پریمیوم داریم. 
توی این آموزش یاد میگیرید که چطوری با استفاده از ابزار Chisel،یک تانل بین 2 سرور ایجاد کنید و ترافیک‌های TCP و UDP را به Ws تبدیل کنید. به عبارت دیگه،نیازی به خرید پلن پریمیوم برای انتقال ترافیک‌های TCP و UDP نیست:)
هدف این آموزش صرفا اشنایی با ابزار Chisel،نحوه استفاده و ترکیب اون با ایده های خودتونه :)))

## 🧐 برای کدوم پروتکل ها کاربردیه؟   
- اکثر پروتکل های بر بستر tcp/udp 
- Wireguard
- V2ray
- SSH 
- OpenVpn





## 🛠️ آموزش کانفیگ سرور دوم - خارج
برای شروع روی پورت 443 یک کانفیگ tcp بسازید و مطمین بشید به درستی کار میکنه
 ترجیحا از ریلیتی یا Tls + Fallback استفاده کنید احتمالا نتیجه بهتری میگیرید😁


برای این تانل بهتره از پورت 80 استفاده کنیم
پس با دستور زیر برسی میکنیم که خالی و قابل استفاده باشه :
```bash
netstat -tulnp 
```


بعد از ساخت کانفیگ برید توی داشبورد کلادفلر و پروکسی کلادفلر رو روی حالت روشن بذارید تا سرورتون پشت کلادفلر مخفی بشه ( ابر باید نارنجی باشه )


   ![](https://preview.redd.it/0qgas702po671.png?width=1035&format=png&auto=webp&s=337e8c7fd09f3e459815cc21465bde7ad018515a)

توی این مرحله بعد از گذشت چند دقیقه کانفیگی که ساختیم از دسترس خارج میشه و دیگه اتصال بهش امکان پذیر نیست که البته منطقیه چون همونطور که گفتیم توی پلن رایگان سرویس های CDN عبور ترافیک های tcp/udp امکان پذیر نیست😂
(البته ریلیتی با هیچ ترنزمیشنی پشت CDN نمیره ولی ما میخوایم ببریمش)

بریم واسه نصب، فقط توجه کنید که این مراحل برای هر 2 سرور یکسان هستش

برای دانلود اخرین نسخه Chisel متناسب با سیستم عاملتون وارد لینک زیر بشید :
```bash
https://github.com/jpillora/chisel/releases
```    


با این دستور یه مسیر جدید میسازیم و واردش میشیم 
```bash
mkdir chisel && cd chisel
```    
 حالا Chisel رو دانلود و اکسترکت میکنیم و سپس بهش مجوز اجرا میدیم 


```bash
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz
```    
     
```bash
gunzip chisel_1.9.1_linux_amd64.gz
```    

     
```bash
chmod +x chisel_1.9.1_linux_amd64
```    

خب کارمون توی سرور خارج تقریبا تموم شده الان میتونید تانل رو اجرا کنید اما قبل از اجرا بهتره یه سرویس براش بنویسیم تا بعد از ریستارت شدن سرور، تانل Chisel مجدد و به طور خودکار اجرا بشه 
پس با دستور زیر اول فایل سرویس رو میسازیم
```bash
sudo nano /etc/systemd/system/chisel.service
```    

حالا کد زیر رو کپی کنید تو فایلی که ساختید سپس دامنه یا ساب دامنه خودتونو بجای YourDomain.com قرار بدید
همونطور که مشخصه پورت 80 داره گوش میکنه و ترافیک های ورودی رو به پورت 443 که پورت کانفیگمون هستش فوروارد میکنه که ترجیها بذارید همین بمونه ولی اگه پورت 80 خالی نیست یا پورت کانفیگتون 443 نیست از این قسمت میتونید تغییرش بدید.
 

```bash
[Unit]
Description=Chisel Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/root/chisel/
ExecStart=/root/chisel/chisel_1.9.1_linux_amd64 server --port 80 --socks5 443 --proxy http://YourDomain.com -v
Restart=on-failure

[Install]
WantedBy=multi-user.target

```    
خب حالا فایل سرویس رو ذخیره کنید و ازش خارج بشید و با دستورات زیر سرویسی که ساختیم رو Enable و Start کنیم 

```bash
sudo systemctl enable chisel
```    
```bash
sudo systemctl start chisel
```    
حالا با دستور زیر وضعیت سرویس Chisel رو برسی میکنیم
توجه کنید که مراحل برای هر 2 سرور تا اینجای کار یکسان هستش و فقط فایل سرویس سرور داخلی (ایران) یکم تفاوت داره که در ادامه میپردازیم بهش...

```bash
sudo systemctl status chisel
```    
اگر تمام مراحل رو به درستی انجام داده باشید الان باید تانل Chisel اکتیو و سبز رنگ باشه و لاگ خروجی چیزی شبیه به اینه که یعنی کارمون توی سرور خارج تموم شده😍

```bash
 chisel.service - Chisel Service
     Loaded: loaded (/etc/systemd/system/chisel.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-12 18:01:56 UTC; 1 day 13h ago
   Main PID: 713 (chisel_1.9.1_li)
      Tasks: 13 (limit: 9259)
     Memory: 656.7M
     CGroup: /system.slice/chisel.service
             └─713 /root/chisel/chisel_1.9.1_linux_amd64 server --port 80 --socks5 443 --proxy http://YourDomain.com -v

Dec 12 18:01:56 debian-8gb-hel1-5 systemd[1]: Started Chisel Service.
Dec 12 18:01:56 debian-8gb-hel1-5 chisel_1.9.1_linux_amd64[713]: 2023/12/12 18:01:56 server: Fingerprint ZQ9dw6O26Sg+EYDccGxUbS6cNsH/>
Dec 12 18:01:56 debian-8gb-hel1-5 chisel_1.9.1_linux_amd64[713]: 2023/12/12 18:01:56 server: Listening on http://0.0.0.0:80
```    

اگر خواستید تغییری توی تنظیمات بدید باید فایل سرویس رو مجدد ویرایش کنید و سرویس Chisel رو با دستور زیر ریستارت کنید 

```bash
sudo systemctl restart chisel
```    


## 🛠️ آموزش کانفیگ سرور اول - ایران    

خب رسیدیم به قسمت جذاب داستان😍
به سرور ایرانتون ssh بزنید و مراحل دانلود و کانفیگ رو دقیقا مثل سرور خارج انجام بدید و پیش برید، فایل سرویس هم دقیقا مثل قبلی میسازیم اما این بار از کد های زیر برای فایل سرویسمون استفاده میکنیم

```bash
[Unit]
Description=Chisel Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/root/chisel/
ExecStart=/root/chisel/chisel_1.9.1_linux_amd64 client http://YourDomain.com 8000:127.0.0.1:443

Restart=on-failure

[Install]
WantedBy=multi-user.target

```
  توی فایل سرویس بالا هم بجای  Yourdomain.com دامنه خودتون رو قرار بدید، اون 8000 هم پورت تانلتون هستش که بعدا باید برای اتصال به کانفیگمون ازش استفاده کنیم و 443 هم همونطور که مشخصه پورت کانفیگ سرور خارجمون هستش که اگر پورت کانفیگتون فرق میکنه باید از اینجا هم تغییرش بدید😊     
حالا سرویس Chisel رو فعال و استارت میکنیم و وضعیتشو برسی میکنیم 

```bash
sudo systemctl enable chisel
```    
```bash
sudo systemctl start chisel
```    
```bash
sudo systemctl status chisel
```    
  اگه تمام مراحل رو درست انجام داده باشید باید سرویس Chisel اکتیو و خروجی چیزی شبیه به این باشه که یعنی تانلمون با موفقیت برقرار شده😋
```bash
● chisel.service - Chisel Service
     Loaded: loaded (/etc/systemd/system/chisel.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-12-14 11:52:29 +0330; 8s ago
   Main PID: 49047 (chisel_1.9.1_li)
      Tasks: 15 (limit: 28657)
     Memory: 156.0M
        CPU: 2.685s
     CGroup: /system.slice/chisel.service
             └─49047 /root/chisel/chisel_1.9.1_linux_amd64 client http:/YourDomain.com 8000:127.0.0.1:443

Dec 14 11:52:29 zxc13 systemd[1]: Started Chisel Service.
Dec 14 11:52:29 zxc13 systemd-journald[466]: Suppressed 3134 messages from chisel.service
Dec 14 11:52:29 zxc13 chisel_1.9.1_linux_amd64[49047]: 2023/12/14 11:52:29 client: Connecting to ws://YourDomain.com:80
Dec 14 11:52:29 zxc13 chisel_1.9.1_linux_amd64[49047]: 2023/12/14 11:52:29 client: tun: proxy#8000=>443: Listening
Dec 14 11:52:33 zxc13 chisel_1.9.1_linux_amd64[49047]: 2023/12/14 11:52:33 client: Connected (Latency 393.753266ms)

```    

برای ریستارت کردن سرویس بعد از تغییرات جدید از دستور زیر استفاده میکنیم 
```bash
sudo systemctl restart chisel
```    
 و برای غیرفعال کردن تانل Chisel در هر 2 سرور از دستور زیر استفاده میکنیم 
 
```bash
sudo systemctl stop chisel 
sudo systemctl disable chisel 
```    



## 🤔 حالا چطوری میتونیم به کانفیگمون متصل شیم؟
خب اینجا دقیقا همون جایی هستش که گفتم باید Chisel رو با ایده های خودتون ترکیب کنید!
مثلا من 2 تا مثال ساده برای درک و شناخت بهتر براتون میزنم که میتونید ازشون استفاده کنید یا ایده بگیرید:)

- تانل Forwarder 
- تانل Site-To-Site (پیشنهادی)

        
