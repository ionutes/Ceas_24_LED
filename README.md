# Ceas pe inel de 24 LED-uri adresabile (WS2812B)

**Realizat de:**
* DRAGOTONIU Ionuț-Constantin (431C)
* BENGULESCU Vlad-Alexandru (431C)

---

## 1. Modul de implementare
Proiectul constă într-un ceas afișat pe un inel cu 24 de LED-uri adresabile individual (WS2812B), plus un LED THT central care funcționează drept secundar (clipește la fiecare secundă care trece).

Sistemul este controlat de un microcontroller **STM32F103** (rulând la 72MHz), iar ora și minutul se pot sincroniza trimițând date prin terminalul serial (conectat prin USB to TTL).

**Logica de funcționare și I/O:**
* **Afișare Ore/Minute:** Cele 24 de LED-uri de pe inel sunt împărțite alternativ. LED-urile cu index par afișează ora (12 poziții), iar minutele sunt împărțite proporțional pe toate cele 24 de leduri.
* **Afișare Secunde:** Secundarul este afișat pe LED-ul THT central.
* **Iluminare și Culori:** LED-urile "stinse" sunt menținute la o culoare de intensitate foarte mică (alb 'dim') pentru fundal. Markerii de timp au culori intense (Roșu pentru oră, Albastru pentru minute, Magenta pentru suprapunere).

---

## 2. Componente Hardware (BOM)
* 1x Microcontroller **STM32F103C8T6**
* 1x Inel cu 24 LED-uri **WS2812B**
* 1x Convertor USB to TTL **CH340G**
* 1x LED THT (secundar) + LED RGB SMD (onboard)
* 1x Condensator **1000uF/35V** (pentru alimentarea stabilă a inelului LED)
* 4x Butoane 6x6x6
* Fire, pini de conexiune, condensatoare SMD și rezistențe (incluzând 220Ω și 330Ω pentru protecția datelor și a LED-urilor).

---

## 3. Descrierea Software
### Resurse și Periferice
* **Porturi I/O:**
  * `PA6` (High Speed): Semnalul de date (`DIN`) către inelul WS2812B.
  * `PA5`: Comanda LED-ului THT secundar.
* **Timp și Temporizări:** Se folosește timerul `SysTick` (la 1ms) pentru numărarea timpului neblocant și `DWT` (Data Watchpoint and Trigger) pentru numărarea exactă a ciclurilor de procesor.
* **UART (USART1):** Configurat la 9600 baud, preluând ora trimisă din terminal prin întreruperi (`HAL_UART_Receive_IT`).

### Arhitectura și Driverul WS2812B
* **Bit-Banging & Inline Assembly:** Semnalele de date extrem de rapide (ordinul nanosecundelor) necesare pentru protocolul WS2812B sunt generate direct pe pinul `PA6` folosind macro-uri de asamblare cu instrucțiuni `nop`. Astfel se evită overhead-ul compilatorului C și se respectă timpii exacți.
* **Zona Critică:** În timpul trimiterii datelor (72 bytes de culoare) către inelul LED, întreruperile sunt temporar oprite folosind `__disable_irq()` și repornite cu `__enable_irq()`. Acest lucru previne întreruperea pachetului de date de către `SysTick` sau UART, garantând afișarea corectă a culorilor.
* **Super-Loop Neblocant:** Arhitectura software citește caractere de la UART și numără secundele fără să folosească funcții blocante (ca `HAL_Delay`), asigurând fluența animațiilor și răspunsul instantaneu la comenzi.

### Amprenta de Memorie
Programul este puternic optimizat, folosind array-ul principal `inel_leds[24][3]` ca framebuffer (72 bytes). 
Dimensiuni finale raportate de compilator:
* **Flash (Program Size):** ~28.4 KB (22% din capacitate)
* **RAM (Data Size):** ~7.0 KB (35% din capacitate)

---

## 4. Instrucțiuni de Utilizare
Pentru a actualiza ora afișată de ceas:
1. Conectați modulul USB to TTL la PC.
2. Deschideți un terminal serial (ex: PuTTY, Serial Monitor din Visual Studio) configurat pe portul COM corespunzător, la un baud rate de **9600**.
3. La pornire, se va afișa mesajul de întâmpinare. Introduceți ora dorită în formatul `HH:MM` (ex: `14:30`) și apăsați Enter.

---
**Bibliografie:**
* Datasheet STM32F103C8T6 - STMicroelectronics
* Datasheet WS2812B Intelligent control LED - Worldsemi
