# Layered Leak Detection: From Point Sensors to Flow Analytics and Pressure Decay

**A whole-house water and heating leak-detection strategy built in layers, the way a BMS combines point detection, flow monitoring and pressure analysis. Each layer catches a different kind of leak: a sudden flood, a dripping tap or passing valve, a slow weep in a sealed heating system, or a safety-valve discharge from an unvented cylinder. This repo ties together monitoring documented across the water and heating projects.**

[![YouTube](https://img.shields.io/badge/YouTube-The%20Control%20Cabinet-red?logo=youtube)](https://www.youtube.com/channel/UC5Cupmz7vlfcodpCX7k8iuQ)
[![Portfolio](https://img.shields.io/badge/Portfolio-the--control--cabinet-24292f?logo=github)](https://github.com/the-control-cabinet)
![Domain](https://img.shields.io/badge/BMS-leak%20detection-blue)
![Status](https://img.shields.io/badge/Status-Partly%20live%20%E2%80%A2%20layers%20in%20progress-orange)

> **The short version:** no single sensor catches every leak, so five layers work together:
> 1. **Point sensors** where water appears
> 2. **Mains flow analytics** for leaks you can't see
> 3. **Sealed-system pressure decay** for heating weeps
> 4. **Safety-device discharge** at the cylinder tundishes
> 5. **Circuit isolation** to locate a leak
>
> Layers 1, 2 and 4 are live; 3 is calibrated and gathering a baseline; 5 is planned.
>
> **Layer 4 in practice:** the tundish sensors caught a weeping relief valve; new expansion vessels fixed it (11 Nov), and the same sensors plus the inlet gauges confirmed the fix (no discharges, 3.8 bar reheat peak).

**Portfolio position:** Automation level · leak detection and alarm strategy. Related: [Mains water metering](https://github.com/the-control-cabinet/mains-water-metering-modbus-rtu) · [Heating plant monitoring](https://github.com/the-control-cabinet/heating-plant-monitoring-esphome)

---

## 1. The layers

| Layer | What it detects | How | Speed | Status |
|---|---|---|---|---|
| **1. Point sensors** | Water on the floor (appliances, sinks, plant areas) | Zigbee water-leak sensors at wet locations | Seconds | ✅ Live |
| **2. Mains flow analytics** | Hidden leaks, dripping taps, passing toilet valves, a burst pipe | Clamp-on ultrasonic meter: **steady low flow for 10 min** alert, per-draw tracking, overnight usage | Minutes to hours | ✅ Live |
| **3. Sealed-system pressure decay** | Slow weeps in the heating primary circuit | Boiler pressure transducer: a daily same-condition reading → **bar/day decay**; low-pressure alarm | Days | 🚧 Sensor installed; calibration and analytics in progress |
| **4. Safety-device discharge** | Unvented cylinder relief discharge (expansion or overpressure) | **Tundish drip sensors** (pulsed plates) + cylinder inlet pressure rise | Seconds | ✅ Live: both tundishes water-tested (naming confirmed); inlet gauges live with alerts (> 4.5 bar); no discharges since the vessels were replaced (11 Nov) |
| **5. Circuit isolation** | *Where* the heating leak is | Isolate circuits one at a time; a proposed **UFH manifold pressure** sensor for a split-system test | Days | 📐 Planned |

```mermaid
flowchart TB
    L1["1 · Point sensors<br/>(water on floor)"] --> A["Alarm handling<br/>(priority, notify, log)"]
    L2["2 · Flow analytics<br/>(steady low flow, overnight use)"] --> A
    L3["3 · Pressure decay<br/>(bar/day, low pressure)"] --> A
    L4["4 · Tundish discharge<br/>+ cylinder overpressure"] --> A
    L5["5 · Circuit isolation<br/>(locate)"] -.diagnosis.-> L3
    A --> R["Response: notify → investigate →<br/>(planned) automatic mains shut-off"]
```

## 2. Alarm priorities

| Priority | Condition | Response |
|---|---|---|
| **P1 (immediate)** | Point sensor wet · tundish discharge · very high continuous flow (burst) | Phone alert now; (planned) close the mains valve on a burst |
| **P2 (same day)** | Steady low flow for 10 min · cylinder inlet pressure > 5.0 bar · boiler pressure < 1.0 bar | Phone alert; investigate |
| **P3 (trend)** | Boiler pressure decay above baseline · rising overnight usage | Dashboard / weekly review |

## 3. Current case: a slow heating leak
- The primary circuit loses about **0.25 bar a week** (1.5 → 1.0 bar over two weeks).
- The boiler's own expansion vessel has been ruled out.
- **Plan:** a daily cold-condition reading for the decay rate, then isolate circuits one at a time, and check the PRV discharge, auto air vents and joints. Details: [heating plant repo, doc 04](https://github.com/the-control-cabinet/heating-plant-monitoring-esphome/blob/main/docs/04-pressure-and-leak-diagnosis.md).

## 4. Lessons
- **Point sensors catch only what reaches them.** Flow and pressure analytics catch what you can't see.
- **Compare like with like.** Heating pressure varies with temperature, so the decay is measured at the same condition each day.
- **Safety-valve discharge is a symptom.** A tundish drip points to an expansion or overpressure problem upstream.

## 5. Roadmap
- [ ] Complete layers 3-5 (calibration, tundish sensors, cylinder inlet pressures, manifold pressure).
- [ ] **Automatic mains shut-off valve** (motorised), triggered by P1 burst/flood conditions, with a manual override.
- [ ] A unified leak dashboard and an alarm-priority implementation (feeds the alarm and trend strategy project).

## 6. Skills demonstrated

| Skill | Evidence |
|---|---|
| Layered detection design | Five complementary layers, each matched to a leak type |
| Alarm prioritisation | P1-P3 with defined responses |
| Analytics | Steady-flow detection, same-condition pressure decay |
| Diagnosis method | Circuit isolation and split-system pressure testing |

📺 [The Control Cabinet on YouTube](https://www.youtube.com/channel/UC5Cupmz7vlfcodpCX7k8iuQ)

*Built and documented by James · part of [The Control Cabinet BMS portfolio](https://github.com/the-control-cabinet). MIT Licence.*

## Update, November 2026: layer 4 in service

- **Tundish sensors commissioned** (5 Nov) on both unvented cylinders: pulsed plates (150 ms every 5 s) through 2N7000 MOSFETs, with two sensitivities: **wet** (2.2 MΩ pull-down, droplets) and **running** (the ESP32's internal pull-down, a flow). Phone alert on wet.
- **It earned its place immediately:** it caught the **expansion relief valve weeping** on reheats, the classic sign of a vessel that has lost its pre-charge. Both vessels and relief valves were replaced on 11 Nov.
- **Commissioning lessons:** at 2 MΩ, every surface matters. A dry gate reading of 0.9 V was traced to a **damaged MOSFET** (replaced), then to flux and handling residue (IPA clean: 30 mV); the plates mount steeper and the cables come in from below so droplets clear.
- **Second signal for the same layer:** cylinder **inlet pressure** gauges (zeroed 11 Nov). A rise towards the 6 bar relief setting during a reheat warns of vessel failure **before** anything discharges.
- Details: [heating plant monitoring, docs 05 and 13](https://github.com/the-control-cabinet/heating-plant-monitoring-esphome).

## Update, 11 Nov 2026: the fix confirmed, and two lessons

- **Root cause fixed:** both expansion vessels replaced (24 L, 3.5 bar). On the first reheats the inlet pressures rose to a clear "knee" at the pre-charge and peaked at **3.8 bar** (relief at 6 bar), and neither tundish registered a wet event. Layer 4 detected the fault, and now confirms the fix.
- **Water test:** wetting the right plate set *Right Wet* and *Right Running*. The test water then pooled under the plate in its printed holder for over 30 minutes (still reading "running"): a sensor holder must **drain**, or one drip looks like a long discharge. Holder to be remounted steeper, cable from below, with a drain hole.
- **A sensing fault can look like a real one:** a loose divider leg in a gauge box made one cylinder read 5 bar; the pin voltage being exactly 1.5× its neighbour's identified the divider rather than the water. Plausibility checks (F20) and comparing like-for-like sensors catch this class of fault.
