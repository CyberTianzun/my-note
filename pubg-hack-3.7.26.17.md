---
title: PUBG External Support for 3.7.26.17
date: 2018-3-30
tags: "hack"
categories: "hack"
---


## code


```cpp
#pragma once

/*
precomp.h:
#pragma once
#define WIN32_LEAN_AND_MEAN
#include <Windows.h>
#include <malloc.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
*/
#include "precomp.h"

typedef uint64_t(*decrypt_func)(uint64_t);

struct tsl {
  union {
    decrypt_func func;
    uint8_t *arr;
  };
};

int tsl_init(struct tsl *tsl);
void tsl_finit(struct tsl *tsl);
uint64_t tsl_decrypt_world(struct tsl *tsl, uint64_t world);
uint64_t tsl_decrypt_actor(struct tsl *tsl, uint64_t actor);
uint64_t tsl_decrypt_prop(struct tsl *tsl, uint64_t prop);

int tsl_init(struct tsl *tsl) {
  tsl->func = (decrypt_func)VirtualAlloc(NULL, 0x600, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
  if (tsl->func == NULL) {
    return 0;
  }
  return 1;
}

void tsl_finit(struct tsl *tsl) {
  if (tsl->func != NULL) {
    VirtualFree(tsl->func, 0, MEM_RELEASE);
    tsl->func = NULL;
  }
}

// ida

#define BYTEn(x, n) (*((BYTE *)&(x) + n))
#define WORDn(x, n) (*((WORD *)&(x) + n))
#define DWORDn(x, n) (*((DWORD *)&(x) + n))

#define IDA_LOBYTE(x) BYTEn(x, 0)
#define IDA_LOWORD(x) WORDn(x, 0)
#define IDA_LODWORD(x) WORDn(x, 0)
#define IDA_HIBYTE(x) BYTEn(x, 1)
#define IDA_HIWORD(x) WORDn(x, 1)
#define IDA_HIDWORD(x) DWORDn(x, 1)

#define BYTE1(x) BYTEn(x, 1)
#define BYTE2(x) BYTEn(x, 2)
#define WORD1(x) WORDn(x, 1)
#define DWORD1(x) DWORDn(x, 1)

// rotate

static uint8_t rol1(uint8_t x, unsigned int count) {
  count %= 8;
  return (x << count) | (x >> (8 - count));
}

static uint16_t rol2(uint16_t x, unsigned int count) {
  count %= 16;
  return (x << count) | (x >> (16 - count));
}

static uint32_t rol4(uint32_t x, unsigned int count) {
  count %= 32;
  return (x << count) | (x >> (32 - count));
}

static uint64_t rol8(uint64_t x, unsigned int count) {
  count %= 64;
  return (x << count) | (x >> (64 - count));
}

static uint8_t ror1(uint8_t x, unsigned int count) {
  count %= 8;
  return (x << (8 - count)) | (x >> count);
}

static uint16_t ror2(uint16_t x, unsigned int count) {
  count %= 16;
  return (x << (16 - count)) | (x >> count);
}

static uint32_t ror4(uint32_t x, unsigned int count) {
  count %= 32;
  return (x << (32 - count)) | (x >> count);
}

static uint64_t ror8(uint64_t x, unsigned int count) {
  count %= 64;
  return (x << (64 - count)) | (x >> count);
}

// macro for uc!

// bool read_size(uint64_t src, void *dest, size_t size)
#define READ(src, dest, size) mem->read_size(src, dest, size)
// template<typename T> read(uint64_t addr)
#define READ32(addr) mem->read<uint32_t>(addr)
#define READ64(addr) mem->read<uint64_t>(addr)

#define GET_ADDR(addr) (g_base_addr + addr)

// credit: https://www.unknowncheats.me/forum/members/2235736.html

static uint32_t get_func_len(struct tsl *tsl, uint64_t func, uint8_t end) {
  uint8_t buf[0x100];
  if (READ(func, buf, sizeof(buf))) {
    if (buf[0] == 0x48) {
      uint32_t len = 0;
      for (; len < sizeof(buf); len++) {
        if (buf[len] == end) {
          return len;
        }
      }
    }
  }
  return 0;
}

static int make_decrypt_func(struct tsl *tsl, uint64_t func) {
  if (!READ(func, tsl->func, 9)) {
    return 0;
  }
  if (tsl->arr[0] != 0x40) {
    return 0;
  }
  uint64_t x = (func + 14) + READ32(func + 10);
  uint32_t len = get_func_len(tsl, x, 0xc3);
  if (!len) {
    return 0;
  }
  if (!READ(x, (char *)tsl->func + 9, len)) {
    return 0;
  }
  if (!READ(func + 14, (char *)tsl->func + 9 + len, 0x80)) {
    return 0;
  }
  return 1;
}

// exports

#define TABLE 0x3c53120

struct uint128_t {
  uint64_t low;
  uint64_t high;
};

uint64_t tsl_decrypt_world(struct tsl *tsl, uint64_t world) {
  struct uint128_t xmm;
  if (!READ(world, &xmm, 16)) {
    return 0;
  }
  uint32_t index = (uint32_t)xmm.low;
  uint16_t x;
  uint16_t y;
  uint8_t z;
  uint8_t w;
  uint8_t q;
  uint64_t e;
  if (index & 1) {
    x = rol2(index, 95);
  }
  else {
    x = ror2(index, 95);
  }
  y = x ^ (ror2(WORD1(index) + 61, -61) + 38223);
  z = x ^ (ror2(WORD1(index) + 61, -61) + 79);
  if (((uint8_t)x ^ (uint8_t)(ror2(WORD1(index) + 61, -61) + 79)) & 1) {
    w = rol1(z, -41);
  }
  else {
    w = ror1(z, -41);
  }
  q = w ^ ((uint8_t)(BYTE1(y) + 71) + 162);
  if (index & 2) {
    e = xmm.high - index;
  }
  else {
    e = index + xmm.high;
  }
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * (q % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(~e);
  memset(tsl->func, 0, 0x600);
  return ror8(ret, 19);
}

uint64_t tsl_decrypt_actor(struct tsl *tsl, uint64_t actor) {
  struct uint128_t xmm;
  if (!READ(actor, &xmm, 16)) {
    return 0;
  }
  uint32_t index = (uint32_t)xmm.low;
  uint16_t x = (uint16_t)~((~(uint16_t)index - 26) ^ 0x1A) ^ ((uint16_t)(IDA_HIWORD(index) + 14) + 7866);
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * (((uint8_t)~((~(~((~(uint8_t)index - 26) ^ 0x1A) ^ (BYTE2(index) - 56)) + 22) ^ 0xEA) ^ (rol1(BYTE1(x), 118) + 12)) % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(index + rol8(index + xmm.high, index & 7));
  memset(tsl->func, 0, 0x600);
  return ror8(ret, -46);
}

uint64_t tsl_decrypt_prop(struct tsl *tsl, uint64_t prop) {
  struct uint128_t xmm;
  if (!READ(prop, &xmm, 16)) {
    return 0;
  }
  uint32_t index = (uint32_t)xmm.low;
  uint16_t x;
  uint16_t y;
  uint16_t z;
  uint8_t w;
  uint16_t q;
  uint8_t e;
  uint16_t r;
  x = index >> 16;
  if (index & 0x10000) {
    y = rol2(x, -25);
  }
  else {
    y = ror2(x, -25);
  }
  z = ror2(index + 45, -45) ^ (y + 37123);
  w = z + 21;
  q = z >> 8;
  e = ror1(w, -21);
  if (q & 4) {
    r = ~(~(uint8_t)q - 303);
  }
  else {
    IDA_LOBYTE(r) = q - 54;
  }
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * ((e ^ ((uint8_t)r + 58)) % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(~(~xmm.high - index));
  memset(tsl->func, 0, 0x600);
  return ror8(ret, -9);
}
```


## Usage:


```cpp

// 3.7.26.17
 
struct tsl tsl;

if (!tsl_init(&tsl)) {
  // what?
}

uint64_t uworld = tsl_decrypt_world(&tsl, g_base_addr + 0x41af780);
uint64_t level = tsl_decrypt_prop(&tsl, uworld + 0x30);
uint64_t game_inst = READ64(uworld + 0x148);
uint64_t local_player = tsl_decrypt_prop(&tsl, READ64(game_inst + 0x38));
uint64_t player_controller = tsl_decrypt_prop(&tsl, local_player + 0x30);
uint64_t player_camera_manager = READ64(player_controller + 0x498);
uint64_t viewport_client = READ64(local_player + 0x60);
uint64_t pworld = READ64(viewport_client + 0x80);

uint64_t actor = tsl_decrypt_actor(&tsl, level + 0xa0);
uint64_t actor_list = READ64(actor);
uint32_t actor_count = READ32(actor + 0x8);

uint64_t local_player_actor = tsl_decrypt_prop(&tsl, player_controller + 0x470);

uint64_t gnames = READ64(g_base_addr + 0x3f488f8);

tsl_finit(&tsl);
```


---


provided by @modnation
