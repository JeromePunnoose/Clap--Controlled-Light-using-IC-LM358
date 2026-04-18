# 🔊 Smart Clap-Controlled Led Light


## 📌 Abstract
A sound-activated LED toggle system that turns a light ON/OFF 
with each clap, built using only analog and digital ICs — 
no microcontroller needed. Operates on a single 9V battery.

---

## ⚙️ Components Used
| Component | Function |
|-----------|----------|
| LM358N Op-Amp | Amplifier + Comparator |
| NE555P Timer | Monostable pulse shaping |
| CD4013 Flip-Flop | Toggle ON/OFF |
| Electret Mic | Sound detection |


## 🔧 Working Principle

### 1. 🎙️ Microphone Signal Generation
When a clap occurs, the electret condenser microphone detects 
the sound wave and converts it into a small alternating voltage 
of typically **1–10 mV**. This weak signal is passed through a 
**0.1 µF coupling capacitor** which blocks DC and allows only 
the AC audio signal to reach the amplifier.

---

### 2. 📈 Non-Inverting Amplifier (LM358N - Section A)
The first half of the LM358N dual op-amp is configured as a 
**non-inverting amplifier**. The microphone signal is applied 
to the **non-inverting input (+IN, Pin 3)**.

The voltage gain is given by:

**Av = 1 + (Rf / Rin) = 1 + (470kΩ / 1kΩ) = 471**

This raises the microphone's millivolt-level signal to several 
hundred millivolts — strong enough to reliably trigger the 
comparator stage.

**Why non-inverting?**
- The output is in phase with the input
- High input impedance — does not load or disturb the 
  microphone signal
- Stable gain set purely by the resistor ratio Rf/Rin

---

### 3. ⚖️ Inverting Comparator (LM358N - Section B)
The second half of the LM358N is configured as an 
**open-loop inverting comparator** (no feedback resistor).

- The amplified signal is applied to the **inverting input 
  (−IN, Pin 6)**
- A fixed reference voltage set by a **100kΩ and 10kΩ voltage 
  divider** is applied to the **non-inverting input (+IN, Pin 5)**

**How it works:**
- When **no clap** → amplified signal < reference voltage → 
  comparator output stays **HIGH**
- When **clap detected** → amplified signal exceeds reference → 
  comparator output switches to **LOW**

This produces a **negative-going (LOW) pulse** on each clap.

**Why inverting comparator?**
- The NE555 timer triggers on a **active-LOW signal** 
  (below VCC/3)
- The inverting comparator naturally produces a LOW pulse 
  on clap detection — making it **directly compatible** with 
  the 555 trigger input without any additional logic

---

### 4. ⏱️ NE555 Timer in Monostable Mode
The NE555P is wired in **monostable (one-shot) mode**. 

- When the comparator's LOW pulse reaches **Pin 2 (TRIG)** 
  of the 555, it triggers a single output pulse
- The output **(Pin 3)** goes **HIGH** for a fixed duration:

**T = 1.1 × R × C = 1.1 × 47kΩ × 10µF ≈ 0.52 seconds**

- After 0.52s, the output automatically returns **LOW**
- During this window, **any echo or noise pulses are ignored** 
  since the 555 is locked in its timing cycle

**Why monostable?**
- A single clap can produce multiple pulses due to echoes
- Without the 555, the flip-flop would toggle multiple times 
  per clap and return to its original state
- The 0.52s clean pulse ensures **exactly one rising edge** 
  reaches the flip-flop per clap ✅

---

### 5. 🔁 CD4013 D Flip-Flop Toggle Operation
The CD4013 D flip-flop is wired in **toggle mode** by 
connecting the **Q̄ (Q-bar) output back to the D input**.

**Truth Table in Toggle Mode:**

| Clock Edge | D (= Q̄) | Q (output) | LED |
|------------|---------|-----------|-----|
| 1st Rising ↑ | 1 | HIGH | ON 💡 |
| 2nd Rising ↑ | 0 | LOW | OFF ⚫ |
| 3rd Rising ↑ | 1 | HIGH | ON 💡 |

**How it works:**
- The **rising edge** of the NE555 output pulse clocks the 
  flip-flop via **Pin 3 (CLK)**
- Since Q̄ is fed back to D, whatever state Q is in, D holds 
  the **opposite value**
- On each rising clock edge, Q **flips to the opposite state**
- This means each clap **toggles the LED** between ON and OFF

The Q output drives the LED through a **1kΩ current-limiting 
resistor**, giving a safe forward current of:

**I = (9V − 2V) / 1kΩ ≈ 7mA** ✅




