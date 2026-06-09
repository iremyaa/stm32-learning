# Day 02 - RCC and Nucleo Board Architecture

## RCC (Reset and Clock Control)

RCC (Reset and Clock Control), STM32'nin saat (clock) ve güç yönetiminden sorumlu birimidir.

STM32 içerisinde bulunan tüm çevresel birimler çalışmak için clock sinyaline ihtiyaç duyar.

Örnek:

* GPIO
* UART
* SPI
* I2C
* Timer
* ADC

Clock olmadan bu donanımlar çalışamaz.

Örneğin GPIOA'nın clock'u kapalıysa:

```c
HAL_GPIO_WritePin(...)
```

komutu çalışsa bile pin değişmez.

Çünkü ilgili çevresel birim aktif değildir.

---

## Neden Clock Enable Yapılır?

STM32 açıldığında enerji tasarrufu sağlamak amacıyla birçok çevresel birim kapalı gelir.

Kullanılacak modüllerin clock'u yazılım tarafından açılır.

Örnek:

```c
__HAL_RCC_GPIOA_CLK_ENABLE();
```

Bu komut GPIOA portunun clock'unu aktif eder.

Arka planda ilgili RCC registerındaki GPIOA enable biti 1 yapılır.

Böylece:

* PA0
* PA1
* PA2
* ...

pinleri kullanılabilir hale gelir.

---

## Nucleo Board Yapısı

Bir geliştirme kartı iki temel bölümden oluşur.

### 1. ST-Link Bölümü

Bilgisayar USB üzerinden haberleşir.

STM32 ise SWD (Serial Wire Debug) protokolünü kullanır.

Bu nedenle arada bir dönüştürücü gerekir.

Bağlantı zinciri:

Computer → USB → ST-Link → SWD → STM32

Kod yükleme işlemi bu yapı sayesinde gerçekleşir.

Bu nedenle Nucleo kartlarda harici programlayıcı satın almaya gerek yoktur.

---

### 2. STM32 MCU Bölümü

Asıl mikrodenetleyici kısmıdır.

İçerisinde:

* GPIO
* UART
* SPI
* I2C
* ADC
* Timer
* Flash Memory
* RAM

bulunur.

---

## User Button

Nucleo kart üzerindeki mavi butondur.

PC13 pinine bağlıdır.

Buton bırakıldığında:

PC13 = 1

Butona basıldığında:

PC13 = 0

Bu yapı Pull-Up kullanılarak gerçekleştirilir.

Bu nedenle tasarım:

* Basılı değil → 1
* Basılı → 0

şeklindedir.

Bu yaklaşım Active Low Button olarak adlandırılır.

Gömülü sistemlerde oldukça yaygındır.

---

## Reset Button

Reset butonu NRST (Reset) pinine bağlıdır.

Butona basıldığında:

NRST = 0

olur ve STM32 yeniden başlatılır.

---

## User LED (LD2)

Nucleo kart üzerindeki kullanıcı LED'idir.

PA5 pinine bağlıdır.

Bağlantı:

PA5 → Direnç → LED

Örnek:

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
```

Bu komut PA5 pinini lojik 1 yapar ve LED yanar.

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
```

Bu komut PA5 pinini lojik 0 yapar ve LED söner.

---

## HAL_GPIO_ReadPin()

Örnek:

```c
HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13);
```

Bu komut PC13 pininin mevcut durumunu okur.

Nucleo kartta PC13 kullanıcı butonuna bağlı olduğundan:

* Basılı değil → 1
* Basılı → 0

değeri okunur.

---

## HAL_GPIO_WritePin()

Örnek:

```c
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
```

Bu komut PA5 pinini lojik 1 yapar.

Nucleo kartta PA5 kullanıcı LED'ine bağlı olduğundan LED yanar.

Bu fonksiyon STM32 uygulamalarında en sık kullanılan GPIO çıkış fonksiyonlarından biridir.
