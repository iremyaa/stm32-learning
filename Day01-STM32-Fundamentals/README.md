# Day 01 - STM32 Fundamentals

## Topics Covered

- STM32 Architecture
- Cortex-M4
- HAL Library
- GPIO
- Input Modes
- Output Modes
- Open Drain vs Push Pull
- I2C
- SPI
- UART
- Registers
- Interrupts

# Day 01 - STM32 Fundamentals

## STM32 ve Cortex-M Mimarisi

Fiziksel kartım olmadığı için piyasada yaygın olarak kullanılan STM32F411CEU6 seçildi. Bu çip, Black Pill geliştirme kartlarında kullanılmaktadır.

STM32F411CEU6, ARM Cortex-M4 çekirdeğini kullanır.

ARM firması işlemci çekirdeğini tasarlar. STMicroelectronics ise bu çekirdeği kullanarak STM32 mikrodenetleyicilerini üretir.

Cortex-M serisi mikrodenetleyiciler için geliştirilmiştir.

Seriler:

* Cortex-M0
* Cortex-M3
* Cortex-M4
* Cortex-M7

Numara büyüdükçe performans artar.

Cortex-M4 içerisinde Floating Point Unit (FPU) bulunur. Bu sayede ondalıklı sayılar donanımsal olarak hızlı işlenebilir.

Kullanım alanları:

* Robotik
* Motor kontrolü
* Filtreleme
* Sinyal işleme

---

## HAL Kütüphanesi

HAL (Hardware Abstraction Layer), STM32 donanımını düşük seviyeli register detaylarına girmeden kullanabilmek için geliştirilmiş bir yazılım katmanıdır.

Örnek:

GPIO pinini açmak için onlarca register işlemi yapmak yerine:

HAL_GPIO_WritePin()

fonksiyonu kullanılabilir.

---

## GPIO

GPIO (General Purpose Input Output), mikrodenetleyicinin dış dünya ile haberleşmesini sağlayan pinlerdir.

### Input Modları

#### Input Floating

Pin ne VCC'ye ne de GND'ye bağlıdır.

* Gürültüden etkilenebilir.
* Genellikle tercih edilmez.

#### Input Pull-Up

Pin dahili direnç ile VCC'ye bağlanır.

* Butona basılmazsa = 1
* Butona basılırsa = 0

#### Input Pull-Down

Pin dahili direnç ile GND'ye bağlanır.

* Butona basılmazsa = 0
* Butona basılırsa = 1

#### Analog Mode

Pin ADC tarafından okunur.

Örnek:

* 0V → 0
* 1.65V → yaklaşık 2048
* 3.3V → 4095

(12 bit ADC için)

---

### Output Modları

#### Push-Pull

İki transistor bulunur:

* PMOS
* NMOS

Pin hem lojik 1 hem de lojik 0 üretebilir.

Kullanım alanları:

* LED sürme
* Röle sürme
* Genel dijital çıkışlar

#### Open Drain

Sadece NMOS bulunur.

Pin:

* 0 gönderebilir
* Boşta kalabilir (Hi-Z)

Lojik 1 oluşturmak için Pull-Up direnci gerekir.

Bu yapı özellikle I2C haberleşmesinde kullanılır.

---

## I2C Haberleşmesi

I2C iki hat kullanır:

* SDA → Veri hattı
* SCL → Saat hattı

Yapı:

* Master cihaz
* Slave cihazlar

Her slave cihazın adresi vardır.

Örnek:

* OLED → 0x3C
* EEPROM → 0x50

Master adres gönderir.

Adres eşleşirse slave ACK gönderir.

Adres eşleşmezse NACK oluşur.

I2C'de Open Drain kullanılır çünkü birden fazla cihaz aynı hattı paylaşır.

---

## Register Kavramı

Registerlar STM32'nin kontrol kutularıdır.

Örnek:

* IDR → Pin okuma
* ODR → Pin çıkış verisi
* BSRR → Tek pinin güvenli ve atomik şekilde değiştirilmesi

---

## Polling ve Interrupt

### Polling

Sürekli kontrol mantığıdır.

Örnek:

"Butona basıldı mı?"

sorusu sürekli sorulur.

### Interrupt

Olay gerçekleşince işlemciye haber verilir.

Daha verimlidir.

---

## SPI Haberleşmesi

SPI dört hat kullanır.

* MOSI
* MISO
* SCK
* CS

Özellikleri:

* I2C'den daha hızlıdır.
* Adres kullanmaz.
* Cihaz seçimi CS hattı ile yapılır.

---

## UART Haberleşmesi

UART iki cihaz arasında seri haberleşme sağlar.

Hatlar:

* TX → Gönderme
* RX → Alma

Metin doğrudan gönderilmez.

Karakterler ASCII kodlarına çevrilir.

Örnek:

A → 01000001

Bu bitler TX hattı üzerinden sırayla gönderilir.

# Alternate Function ve GPIO Registerları

## Alternate Function (AF) Nedir?

Normal şartlarda bir GPIO pini CPU tarafından kontrol edilir.

Örneğin LED yakmak istediğimizde CPU, GPIO registerlarını değiştirerek pin seviyesini belirler.

Akış şu şekildedir:

CPU → GPIO → Pin

Ancak STM32 içerisinde yalnızca CPU bulunmaz.

Donanım birimleri:

* UART
* SPI
* I2C
* Timer
* ADC

gibi birçok çevresel birim vardır.

Bazı durumlarda pini CPU yerine bu donanım birimlerinin kontrol etmesi gerekir.

Örneğin UART üzerinden veri gönderilirken CPU'nun TX pinini sürekli açıp kapatması verimsizdir.

Bunun yerine UART modülü gerekli bit dizisini donanım seviyesinde üretir ve doğrudan pine gönderir.

Bu nedenle ilgili pin Alternate Function (AF) moduna alınır.

### Önemli Not

AF modunda CPU tamamen devre dışı kalmaz.

CPU:

* UART ayarlarını yapar
* Baud rate belirler
* Gönderilecek veriyi verir

Fiziksel sinyal üretimini ise UART donanımı gerçekleştirir.

---

## Neden Alternate Function Kullanılır?

Bazı sinyaller çok hassas zamanlama gerektirir.

Örnekler:

* UART baud rate üretimi
* SPI clock üretimi
* PWM sinyali
* I2C haberleşmesi

Bu işlemleri yazılımla yapmak hem işlemciyi yorar hem de zamanlama hatalarına yol açabilir.

Bu nedenle STM32 içerisinde bu işleri donanım seviyesinde yapan özel modüller bulunur.

AF modu sayesinde bu modüller doğrudan pine erişebilir.

---

## GPIO_MODER Registerı

GPIO_MODER registerı pinin hangi modda çalışacağını belirler.

Her pin için 2 bit ayrılmıştır.

STM32 portları:

* PA0 - PA15
* PB0 - PB15
* PC0 - PC15

şeklinde düzenlenmiştir.

Örnek:

* PA0 → Input
* PA1 → Analog
* PA2 → Alternate Function
* PA5 → Output

Görevi:

Pinin kullanım amacını belirlemek.

---

## GPIO_OTYPER Registerı

GPIO çıkış modunda çalışırken çıkışın elektriksel karakteristiğini belirler.

### Push-Pull

Pin hem lojik 1 hem de lojik 0 üretebilir.

İçerisinde:

* PMOS
* NMOS

bulunur.

Kullanım alanları:

* LED sürme
* Röle sürme
* Dijital çıkışlar

---

### Open Drain

Pin yalnızca GND seviyesini üretebilir.

Durumlar:

* 0 → GND
* 1 → Hi-Z

Lojik 1 seviyesi pull-up direnci tarafından oluşturulur.

Özellikle I2C haberleşmesinde kullanılır.

Sebebi:

Birden fazla cihaz aynı veri hattını paylaşabilir.

Open Drain sayesinde cihazlar birbirleriyle çakışmaz.

---

## GPIO_OSPEEDR Registerı

Pinin yükselme ve düşme hızını belirler.

Yani:

* 0V → 3.3V geçiş süresi
* 3.3V → 0V geçiş süresi

Ayarlar:

* Low
* Medium
* High
* Very High

Yüksek hız:

Avantaj:

* Daha hızlı geçiş

Dezavantaj:

* Daha fazla güç tüketimi
* Daha fazla elektromanyetik gürültü (EMI)

Örnek:

* LED uygulaması → Low
* SPI haberleşmesi → High veya Very High

---

## GPIO_PUPDR Registerı

Input modundaki pinlerin boşta kalmasını önler.

Boşta kalan pinler:

* Gürültüden etkilenebilir
* Rastgele 0 veya 1 okunabilir

Çözümler:

### Pull-Up

Pin direnç üzerinden VDD'ye bağlanır.

Boşta:

* 1

### Pull-Down

Pin direnç üzerinden GND'ye bağlanır.

Boşta:

* 0

Özellikle buton uygulamalarında kullanılır.

---

## AFRL ve AFRH Registerları

Bir pini AF moduna almak tek başına yeterli değildir.

STM32'nin hangi çevresel birimin pini kullanacağını da bilmesi gerekir.

Örnek:

PA2 pini:

* USART2_TX olabilir
* Timer çıkışı olabilir
* Başka bir fonksiyona sahip olabilir

Bu seçim Alternate Function numarası ile yapılır.

Örnek:

AF7 → USART2_TX

Bu durumda PA2 pininin kontrolü UART modülüne verilmiş olur.

### Register Dağılımı

Her pin için 4 bit ayrılmıştır.

AFRL:

* Pin 0 - Pin 7

AFRH:

* Pin 8 - Pin 15

Örnek:

* PA2 → AFRL içerisinde ayarlanır
* PA10 → AFRH içerisinde ayarlanır
