---
name: user-ps2-graphics-background
description: User has no PS2 GS/GIF/DMA or graphics-SDK background (2026-11-07); explain hardware from basics
metadata:
  type: user
---

The user said (2026-11-07) they know "literally nothing about the PS2's graphics hardware or SDK"
and found the func_001581E0 explanation (DMA tags, GIF tags, GS registers) a dense read; they
went to research offline.

**How to apply:** when code touches GS/GIF/DMA/VIF/IPU, explain the hardware term first (what a
DMA tag / GIF tag / register is for) before the bit layout; decode constants in a table with field
names and values; point to a Sony sample doing the same thing (`ref/globe` samples, e.g. mpegstr
`disp.c` `setImageTag`). Keep it step by step ([[explain-step-by-step]]), and don't assume SDK
vocabulary. Related: [[sony-sdk-samples]].
