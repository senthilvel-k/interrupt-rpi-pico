
# Interrupt Handling with Raspberry Pi Pico

> **External GPIO interrupt — button press triggers ISR, LED toggles 5× as visual proof.**

[![Language](https://img.shields.io/badge/Language-C%20(C11)-blue?style=flat-square&logo=c)](https://en.cppreference.com/w/c/11)
[![SDK](https://img.shields.io/badge/SDK-Pico%20SDK-brightgreen?style=flat-square)](https://github.com/raspberrypi/pico-sdk)
[![Series](https://img.shields.io/badge/Series-Episode%2002-red?style=flat-square)](#)

---

**[🌐 Project Demo Site](https://senthilvel-k.github.io/interrupt-rpi-pico/)** · **[👤 Portfolio](https://senthilvel-k.github.io/)** · **[📄 Full Report](./docs/Interrupt_Report.pdf)**

---

## Overview

Demonstrates **external interrupt handling** on the RP2040. A button on GPIO 11 is configured to fire an ISR on a rising edge. The interrupt handler toggles the onboard LED 5 times at 1-second intervals — all without polling. The CPU's main loop stays free.

**Key insight:** The CPU is not constantly checking the button. It's doing nothing (or could be doing other work). The hardware detects the voltage edge and forces a context switch to your handler function automatically.

---

## What You'll Learn

- The difference between **polling** (wasteful) and **interrupts** (efficient)
- What a **rising edge** is — and how the RP2040 GPIO hardware detects it
- How to register an **ISR callback** with `gpio_set_irq_enabled_with_callback()`
- CPU **context switching** — saving registers, jumping to ISR, restoring state
- Why `busy_wait_ms()` is used inside ISRs (not `sleep_ms()`)

---

## Hardware

| Component | GPIO Pin | Direction |
|-----------|----------|-----------|
| Onboard LED | GPIO 25 (PICO_DEFAULT_LED_PIN) | OUTPUT |
| Button (EXT) | GPIO 11 | INPUT |

> Button connects GPIO 11 to 3.3V. On press, GPIO 11 goes LOW→HIGH = rising edge.

---

## How It Works

```
Button Press (GPIO 11: LOW→HIGH)
        │
        ▼
  Hardware detects RISING EDGE
        │
        ▼
  CPU suspends main loop
  (saves registers + PC)
        │
        ▼
  interrupt_handler() executes:
    for i in 0..5:
      LED ON  → busy_wait_ms(1000)
      LED OFF → busy_wait_ms(1000)
        │
        ▼
  CPU restores context
  Main loop resumes
```

## Key API

```c
gpio_set_irq_enabled_with_callback(
    EXT_INT_PIN,           // GPIO to watch
    GPIO_IRQ_EDGE_RISE,    // trigger on LOW→HIGH
    true,                  // enable
    &interrupt_handler     // your ISR function
);
```

---

## Build & Flash

```bash
git clone https://github.com/senthilvel-k/interrupt-rpi-pico.git
cd interrupt-rpi-pico && mkdir build && cd build
cmake .. && make -j4
# Hold BOOTSEL, plug in, copy interrupt.uf2
```

---

## Series Context

This is **Episode 02** of the Raspberry Pi Pico Learning Series by Senthil Vel K.

| # | Project | Concept |
|---|---------|---------|
| 01 | [LCD Display](https://github.com/senthilvel-k/lcd-rpi-pico) | 4-bit parallel, HD44780 |
| **02** | **Interrupt Handling** | **GPIO IRQ, ISR, rising edge** |
| 03 | [Ultrasonic Sensor](https://github.com/senthilvel-k/ultrasonic-rpi-pico) | Pulse timing, distance |
| 04 | [Keypad Matrix](https://github.com/senthilvel-k/keypad-rpi-pico) | Matrix scan, pull-ups |
| 05 | [Potentiometer](https://github.com/senthilvel-k/potentiometer-rpi-pico) | ADC, PWM brightness |

> Full portfolio: **[senthilvel-k.github.io](https://senthilvel-k.github.io/)**

---

*Senthil Vel K · skumara4 · Visteon Corporation · October 2023 · MIT License*
