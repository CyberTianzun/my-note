---
title: PUBG External Support for 3.7.26.15
date: 2018-3-24
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
  uint32_t y;
  uint32_t z;
  uint16_t w;
  x = ror2(index + 93, -93);
  y = index >> 16;
  if (index & 0x20000) {
    z = ~(y + 119) + ~(y - 119);
  }
  else {
    z = ~(y ^ 0x77) + y + 120;
  }
  w = x ^ ((uint16_t)z + 33235);
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * ((ror1((x ^ (z - 45)) - 59, 59) ^ (rol1(BYTE1(w) + 21, 21) + 154)) % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(~(~xmm.high - index));
  memset(tsl->func, 0, 0x600);
  return ror8(ret, -121);
}

uint64_t tsl_decrypt_actor(struct tsl *tsl, uint64_t actor) {
  struct uint128_t xmm;
  if (!READ(actor, &xmm, 16)) {
    return 0;
  }
  uint32_t index = (uint32_t)xmm.low;
  uint16_t x = rol2(index + 94, 94) ^ (ror2(WORD1(index), -90) + 2622);
  uint64_t y;
  if (index & 2) {
    y = xmm.high ^ index;
  }
  else {
    y = xmm.high + index;
  }
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * ((((uint8_t)~(~BYTE1(x) + 46) + 4) ^ rol1((rol2(index + 94, 94) ^ (ror2(WORD1(index), -90) + 62)) + 78, 78)) % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(y);
  memset(tsl->func, 0, 0x600);
  return ror8(ret, 70);
}

uint64_t tsl_decrypt_prop(struct tsl *tsl, uint64_t prop) {
  struct uint128_t xmm;
  if (!READ(prop, &xmm, 16)) {
    return 0;
  }
  uint32_t index = (uint32_t)xmm.low;
  uint16_t x = (uint16_t)(index + 23) ^ ((WORD1(index) ^ 0xFF9B) + 32135);
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * (((((index + 23) ^ ((BYTE2(index) ^ 0x9B) - 121)) + 79) ^ ((BYTE1(x) - 63) + 50)) % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(ror8(xmm.high, index & 7) - index);
  memset(tsl->func, 0, 0x600);
  return ror8(ret, 107);
}
```


## Usage:


```cpp

// 3.7.26.15
 
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
