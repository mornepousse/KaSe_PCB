# KaSe V3 "Endgame" Roadmap

This document outlines the planned architectural changes for the next major iteration of the KaSe keyboard - Version 3, code-named "Endgame". This ambitious update represents a significant evolution from the current V2 design, targeting professional-grade performance and rich user interface capabilities.

## Overview

KaSe V3 aims to push the boundaries of custom keyboard design by combining ultra-high polling rates, wireless connectivity, and advanced display capabilities while maintaining manufacturability through JLCPCB assembly.

---

## 1. Core Architecture (MCU)

### Primary MCU: STM32H750VBT6

**Migration from**: ESP32-S3  
**Target MCU**: **STM32H750VBT6**

#### Rationale
- **Raw Processing Power**: 480 MHz ARM Cortex-M7 core provides the computational headroom needed to handle:
  - 8000 Hz USB polling rate with minimal jitter
  - Real-time display rendering for the round LCD
  - Simultaneous wireless communication management
  - Advanced keyboard features and macros

#### Key Constraint
- **Limited Internal Flash**: Only 128 KB on-chip flash memory
- **Solution**: External memory integration (see section 2) using XIP (Execute In Place) architecture

---

## 2. Memory & Storage

### External Flash Integration

**Component**: **W25Q128 (16MB)** or **W25Q64 (8MB)** QSPI Flash

#### Purpose
- **XIP (Execute In Place)**: Store and execute firmware directly from external flash, overcoming the STM32H750's internal flash limitation
- **Asset Storage**: High-resolution images, custom fonts, and graphics for the round display UI
- **Data Logging**: Optional storage for macros, configurations, and usage statistics

#### Interface
- **QSPI (Quad SPI)**: High-speed interface for fast code execution and asset streaming

---

## 3. USB Connectivity (8000Hz)

### USB High-Speed Implementation

**Component**: **Microchip USB3300** External USB PHY

#### Goal
Enable **USB High Speed (480 Mbps)** operation to achieve a stable **8000 Hz polling rate** with 125µs microframe intervals.

#### Why External PHY?
- **Native USB Limitation**: The STM32H750's built-in USB operates at Full Speed (12 Mbps), which limits polling to 1000 Hz maximum
- **Performance Target**: Professional gaming and precision input requires 8000 Hz polling for sub-millisecond response times
- **USB3300 Benefits**:
  - ULPI interface to STM32H7
  - Industry-standard USB High-Speed PHY
  - Proven reliability in high-performance applications

---

## 4. Wireless & Coprocessor

### Dedicated Radio Module

**Component**: **nRF52840** module (Raytac MDBT50Q-1M or Holyiot variant)

#### Role
Dedicated wireless coprocessor operating independently from the main STM32H7, offloading RF management and reducing latency.

#### Supported Protocols

1. **2.4GHz Proprietary (ESB - Enhanced ShockBurst)**
   - **Purpose**: Ultra-low latency wireless communication for mouse receiver mode
   - **Target Latency**: Sub-millisecond wireless response
   - **Use Case**: Wireless gaming mode with custom dongle

2. **Bluetooth Low Energy (BLE)**
   - **Purpose**: Standard wireless connectivity for compatibility with multiple devices
   - **Use Case**: Office/productivity mode, multi-device pairing

#### Interconnection
- **Interface**: High-speed SPI or UART connection to STM32H7 master
- **Architecture**: nRF52840 acts as a wireless peripheral, with the STM32H7 handling all keyboard logic and USB communication

---

## 5. Display & UI

### Round LCD Display

**Component**: **GC9A01** controller (1.28" Round LCD, 240×240 pixels)

#### Display Interface
- **SPI with DMA**: Hardware-accelerated display updates managed by STM32H7's DMA controller
- **Performance**: Smooth UI rendering without impacting keyboard scanning or USB polling

#### Touch Input

**Component**: **CST816S** capacitive touch controller

- **Interface**: I2C
- **Purpose**: Interactive UI navigation, settings adjustment, and visual feedback
- **Features**: Gesture support, tap/swipe detection

#### UI Capabilities
- Real-time typing statistics
- Layer indicators and visual feedback
- Custom animations and themes
- Configuration menus
- Wireless connection status
- Battery monitoring (in wireless mode)

---

## 6. Manufacturing

### Design for Manufacturing (DFM)

**Target Supplier**: **JLCPCB** assembly service

#### Component Selection Strategy
- **LCSC Library**: All components sourced from JLCPCB's preferred supplier (LCSC)
- **Part Categories**:
  - **Basic Parts**: Prioritize whenever possible (no additional fees, faster assembly)
  - **Extended Parts**: Use when necessary for specialized components (USB3300, GC9A01, nRF52840 module)
- **Minimize Manual Work**: Design to minimize or eliminate hand-soldering requirements

#### Goals
- **Accessibility**: Enable community members to order assembled boards without specialized equipment
- **Cost Efficiency**: Leverage JLCPCB's assembly pricing for small-batch production
- **Quality**: Maintain consistent manufacturing quality through automated assembly

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    KaSe V3 "Endgame"                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐         ┌──────────────┐                │
│  │  STM32H750   │◄───────►│   USB3300    │◄──── USB-C     │
│  │  (480 MHz)   │  ULPI   │  (HS PHY)    │    8000 Hz     │
│  └──────┬───────┘         └──────────────┘                │
│         │                                                   │
│         │ QSPI          ┌──────────────┐                  │
│         ├──────────────►│  W25Q128/64  │                  │
│         │               │  Flash (XIP) │                  │
│         │               └──────────────┘                  │
│         │                                                   │
│         │ SPI+DMA       ┌──────────────┐                  │
│         ├──────────────►│   GC9A01     │                  │
│         │               │  Round LCD   │                  │
│         │               │  240×240 px  │                  │
│         │               └──────────────┘                  │
│         │                                                   │
│         │ I2C           ┌──────────────┐                  │
│         ├──────────────►│   CST816S    │                  │
│         │               │ Touch Ctrl   │                  │
│         │               └──────────────┘                  │
│         │                                                   │
│         │ SPI/UART      ┌──────────────┐                  │
│         └──────────────►│  nRF52840    │◄──── 2.4GHz     │
│                         │  (Wireless)  │    ESB / BLE    │
│                         └──────────────┘                  │
│                                                             │
│  Key Matrix ──► STM32H750 GPIO                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Development Phases

### Phase 1: Core Platform (Q1-Q2 2026)
- [ ] STM32H750 schematic and PCB layout
- [ ] W25Q128 QSPI flash integration
- [ ] USB3300 high-speed USB implementation
- [ ] Basic firmware with XIP support
- [ ] 8000 Hz polling validation

### Phase 2: Wireless Integration (Q2-Q3 2026)
- [ ] nRF52840 module integration
- [ ] SPI/UART communication protocol
- [ ] ESB proprietary wireless protocol
- [ ] BLE standard wireless support
- [ ] Custom receiver dongle design

### Phase 3: Display & UI (Q3-Q4 2026)
- [ ] GC9A01 round display integration
- [ ] CST816S touch controller integration
- [ ] DMA-accelerated graphics pipeline
- [ ] UI framework and themes
- [ ] Interactive configuration system

### Phase 4: Manufacturing Optimization (Q4 2026)
- [ ] Component selection optimization for JLCPCB
- [ ] DFM review and adjustments
- [ ] Assembly documentation
- [ ] Prototype run and validation
- [ ] Production release

---

## Technical Challenges

### High-Speed USB at 8000 Hz
- **Challenge**: Maintaining consistent 125µs polling with zero jitter
- **Approach**: Dedicated USB interrupt priority, optimized USB stack, real-time OS considerations

### XIP Performance
- **Challenge**: Ensuring code execution from external flash doesn't bottleneck performance
- **Approach**: QSPI running at maximum speed, critical code in internal RAM, caching strategies

### Power Management
- **Challenge**: Managing power consumption with high-performance MCU and display
- **Approach**: Dynamic frequency scaling, display sleep modes, wireless power optimization

### Thermal Management
- **Challenge**: 480 MHz STM32H7 + USB3300 + wireless in compact form factor
- **Approach**: Thermal simulation, copper pour optimization, component placement strategy

---

## Open Questions

1. **Battery Integration**: Should V3 include an integrated battery for true wireless operation, or rely on external power?
2. **MCU Package**: LQFP-100 vs LQFP-64 - balancing GPIO availability with board size
3. **Flash Capacity**: 16MB (W25Q128) vs 8MB (W25Q64) - cost vs features trade-off
4. **Wireless Priority**: Primary focus on ESB low-latency or BLE compatibility?

---

## Contributing

This roadmap is a living document. Community feedback and contributions are welcome. Please open an issue or discussion to suggest changes or additions.

---

## References

- [STM32H750VBT6 Datasheet](https://www.st.com/en/microcontrollers-microprocessors/stm32h750vb.html)
- [USB3300 USB PHY Datasheet](https://www.microchip.com/en-us/product/USB3300)
- [nRF52840 Product Specification](https://www.nordicsemi.com/products/nrf52840)
- [GC9A01 Display Controller](https://www.buydisplay.com/)
- [JLCPCB Assembly Service](https://jlcpcb.com/smt-assembly)

---

*Last Updated: 2026-01-08*
